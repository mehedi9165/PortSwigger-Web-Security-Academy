# Reflected XSS into a JavaScript String with Angle Brackets HTML-Encoded

## 1. Vulnerability

**Vulnerability:** Reflected Cross-Site Scripting (Reflected XSS)

**Difficulty:** Apprentice

**Injection Context:** JavaScript string inside an HTML script block

**Source:** Search query parameter

**Sink:** JavaScript execution context

**Payload:**

```javascript
'-alert(1)-'
```

### Description

The application contains a reflected XSS vulnerability in its search query tracking functionality.

The search term is reflected into a JavaScript string within the page. Although angle brackets (`<` and `>`) are HTML-encoded, the application does not correctly handle input that can terminate the JavaScript string.

An attacker can use a single quote (`'`) to escape the existing string and inject a JavaScript expression that calls `alert(1)`.

This demonstrates why HTML encoding alone is insufficient when untrusted data is inserted into JavaScript code.

## 2. Objective

- Identify where the search query is reflected in the response.
- Determine that the input is inside a JavaScript string.
- Understand why HTML encoding does not prevent JavaScript string injection.
- Escape the existing JavaScript string.
- Inject a JavaScript expression that calls `alert(1)`.
- Identify the appropriate defensive measures.

## 3. Step-by-Step Exploitation

### Step 1 — Open the lab

Open the PortSwigger lab and navigate to the blog search functionality.

Locate the search box.

### Step 2 — Submit a random alphanumeric string

Enter a unique test value:

```text
test123abc
```

Submit the search.

A unique value makes it easier to locate the reflection in the HTTP response.

### Step 3 — Intercept the search request

Using Burp Suite:

1. Submit the search through the browser.
2. Locate the request in Proxy HTTP history.
3. Send the request to **Repeater**.
4. Send the request and inspect the response.

Search the response for:

```text
test123abc
```




<img width="1320" height="731" alt="Screenshot 2026-10-15 at 12 01 12 AM" src="https://github.com/user-attachments/assets/8f16921d-c605-4ed2-99ec-785aa87af2db" />

### Step 4 — Identify the JavaScript context

You should find the search term reflected inside a JavaScript string.

For example, the response may contain code similar to:

```html
<script>
    var searchTerms = 'test123abc';
</script>
```


<img width="1321" height="734" alt="Screenshot 2026-10-14 at 11 54 38 PM" src="https://github.com/user-attachments/assets/b553f972-758a-48f1-ab92-5374f44b2df2" />



In inspect mode it looks like below:


<img width="973" height="462" alt="Screenshot 2026-10-14 at 8 47 08 PM" src="https://github.com/user-attachments/assets/6cbc2cd8-378a-4018-a98c-3c9a61a9dd31" />




The important observation is that the input is enclosed by single quotes.



### Step 5 — Submit the XSS payload

Replace the random search value with:

```javascript
'-alert(1)-'
```

Submit the request through Burp Repeater, or use the resulting search URL in the browser.


<img width="1319" height="737" alt="Screenshot 2026-10-14 at 11 56 05 PM" src="https://github.com/user-attachments/assets/7fb7319c-fc11-4ac8-80cf-e7495041b96b" />


In console:

<img width="974" height="467" alt="Screenshot 2026-10-14 at 8 58 31 PM" src="https://github.com/user-attachments/assets/f5cde5de-6038-415c-8096-a5c37aab3249" />



### Step 7 — Verify the result

Open the crafted URL in the browser.

If the payload executes successfully, a JavaScript alert displaying:

```text
1
```

will appear.

This confirms that the injected expression was executed in the page's JavaScript context.

---

<img width="1317" height="742" alt="Screenshot 2026-10-14 at 11 56 44 PM" src="https://github.com/user-attachments/assets/b420546d-9759-4cc9-a0a4-62bed3d28d5e" />


<img width="1316" height="673" alt="Screenshot 2026-10-15 at 12 08 53 AM" src="https://github.com/user-attachments/assets/91331103-4f86-4487-b769-498dccf5209f" />


## 4. How the Payload Works

Payload:

```javascript
'-alert(1)-'
```

Assume the application generates the following JavaScript:

```javascript
var searchTerms = 'USER_INPUT';
```

With normal input, it becomes:

```javascript
var searchTerms = 'test123abc';
```

With the malicious input, the resulting code becomes conceptually:

```javascript
var searchTerms = ''-alert(1)-'';
```

The browser parses the resulting expression as:

```javascript
'' - alert(1) - ''
```

The empty strings are converted to numeric zero during subtraction. JavaScript evaluates the expression from left to right, and `alert(1)` is called during evaluation.

The alert appears before the remaining expression finishes evaluating.

### Payload breakdown

**First single quote (`'`):** Terminates the application's original JavaScript string.

**Minus sign (`-`):** Joins the injected expression to the surrounding JavaScript using subtraction.

**`alert(1)`:** Calls the JavaScript alert function.

**Final `-'`:** Completes the injected expression and provides a closing quote for the string context.

The critical technique is escaping the string context without needing to introduce a new HTML element.

## 5. Execution Flow

```text
Attacker-controlled search term
             ↓
Server reflects search term
             ↓
Value enters JavaScript string
             ↓
Single quote terminates string
             ↓
Injected JavaScript expression
             ↓
alert(1) executes
```


## 6. Impact

Depending on the application and the victim's privileges, reflected XSS may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify webpage content.
- Perform actions available to the victim.
- Display convincing phishing content.
- Access information exposed to JavaScript.
- Target privileged users.

The actual impact depends on the application's architecture, authentication model, and security controls.

## 7. Defensive Measures

### 7.1 Avoid inserting untrusted data directly into JavaScript

Prefer passing data through mechanisms that preserve the distinction between data and executable code.

For example, use JSON serialization for structured data rather than manually constructing JavaScript strings.

### 7.2 Apply JavaScript-context-safe encoding

If data must be embedded inside JavaScript, use a well-tested encoding mechanism appropriate for that context.

Simply HTML-encoding angle brackets is not sufficient.

### 7.3 Prefer safe DOM APIs

When the goal is to display a search term, use APIs such as:

```javascript
element.textContent = searchTerms;
```

This treats the value as text rather than executable markup.

### 7.4 Use secure templating

Use a framework or templating system that supports contextual output encoding. Avoid concatenating untrusted data into inline script blocks.

### 7.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide additional defense in depth against XSS. It should not replace correct output handling.

