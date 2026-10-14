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

### Step 4 — Identify the JavaScript context

You should find the search term reflected inside a JavaScript string.

For example, the response may contain code similar to:

```html
<script>
    var searchTerms = 'test123abc';
</script>
```

This is an illustrative example. Record the exact JavaScript context observed in your lab response.

The important observation is that the input is enclosed by single quotes.

The data flow is:

```text
Search input
     ↓
HTTP request
     ↓
Server response
     ↓
JavaScript string
```

### Step 5 — Understand the protection limitation

The application HTML-encodes angle brackets:

```text
<  →  &lt;
>  →  &gt;
```

This can prevent certain HTML injection techniques, but the reflection occurs inside JavaScript code.

In this context, the single quote is significant because it can terminate the existing JavaScript string if it is not properly escaped.

### Step 6 — Submit the XSS payload

Replace the random search value with:

```javascript
'-alert(1)-'
```

Submit the request through Burp Repeater, or use the resulting search URL in the browser.

### Step 7 — Verify the result

Open the crafted URL in the browser.

If the payload executes successfully, a JavaScript alert displaying:

```text
1
```

will appear.

This confirms that the injected expression was executed in the page's JavaScript context.

---

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

## 6. Why Does This Work?

The vulnerability exists because the application handles input as if HTML encoding alone were sufficient.

However, HTML parsing and JavaScript parsing are different contexts.

The application places user-controlled input into a JavaScript string without correctly encoding it for that context.

As a result, the attacker can change the structure of the JavaScript code.

**Key lesson:** Output encoding must match the context in which the data is inserted.

## 7. Impact

Depending on the application and the victim's privileges, reflected XSS may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify webpage content.
- Perform actions available to the victim.
- Display convincing phishing content.
- Access information exposed to JavaScript.
- Target privileged users.

The actual impact depends on the application's architecture, authentication model, and security controls.

## 8. Defensive Measures

### 8.1 Avoid inserting untrusted data directly into JavaScript

Prefer passing data through mechanisms that preserve the distinction between data and executable code.

For example, use JSON serialization for structured data rather than manually constructing JavaScript strings.

### 8.2 Apply JavaScript-context-safe encoding

If data must be embedded inside JavaScript, use a well-tested encoding mechanism appropriate for that context.

Simply HTML-encoding angle brackets is not sufficient.

### 8.3 Prefer safe DOM APIs

When the goal is to display a search term, use APIs such as:

```javascript
element.textContent = searchTerms;
```

This treats the value as text rather than executable markup.

### 8.4 Use secure templating

Use a framework or templating system that supports contextual output encoding. Avoid concatenating untrusted data into inline script blocks.

### 8.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide additional defense in depth against XSS. It should not replace correct output handling.

## 9. Remediation Summary

| Issue | Recommended Defense |
|---|---|
| Input reflected into JavaScript | Avoid direct interpolation |
| Single quote can terminate a string | Use context-appropriate encoding |
| HTML encoding used as the only defense | Apply JavaScript-context-safe handling |
| Dynamic script construction | Use safe data serialization |
| Search term displayed in the page | Prefer `textContent` |
| Additional XSS protection needed | Implement CSP |

## 10. Key Takeaway

This lab demonstrates that **HTML encoding does not automatically make a JavaScript string safe**.

The essential data flow is:

```text
Search parameter
      ↓
JavaScript string
      ↓
Single-quote escape
      ↓
Injected expression
      ↓
alert(1)
```

When testing XSS, identify the exact context before choosing a payload.

Ask:

1. Is the input reflected in HTML text, an HTML attribute, a URL, or JavaScript?
2. Which characters are encoded?
3. Can the input terminate the current string or expression?
4. Can the surrounding syntax be used to execute a JavaScript expression?
5. What encoding or design change would prevent the injection?

## 11. Evidence

**Test input:**

```text
test123abc
```

**Observed context:**

```javascript
var searchTerms = 'test123abc';
```

**Payload:**

```javascript
'-alert(1)-'
```

**Trigger:** Load the crafted search URL.

**Expected result:** A JavaScript alert displaying `1`.

**Verification:** Confirm that the reflected input appears inside the JavaScript string and that the injected expression executes.

**Lab status:** Solved.

**Remediation:** Avoid direct JavaScript string interpolation and use context-appropriate output handling.
