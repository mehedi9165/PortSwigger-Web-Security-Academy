This PortSwigger lab demonstrates an important XSS technique: injecting a JavaScript event handler into an HTML attribute when angle brackets (`<` and `>`) are HTML-encoded.

For your GitHub portfolio, document it as follows.

# Reflected XSS into an HTML Attribute with Angle Brackets HTML-Encoded

## 1. Vulnerability

**Vulnerability:** Reflected Cross-Site Scripting (Reflected XSS)

**Difficulty:** Apprentice

**Injection context:** Quoted HTML attribute

**Source:** Search input / HTTP request parameter

**Sink:** HTML attribute rendered in the response

**Payload:**

```
"onmouseover="alert(1)
```

### Description

The application contains a reflected XSS vulnerability in its blog search functionality. User-controlled input is reflected into a quoted HTML attribute, but angle brackets are HTML-encoded.

Although this prevents the input from directly creating new HTML elements, it does not prevent an attacker from escaping the existing quoted attribute and injecting a new event-handler attribute.

The payload uses a double quote (`"`) to terminate the original attribute value and introduces an `onmouseover` event handler. When the user moves the mouse over the affected element, the browser executes `alert(1)`.

## 2. Objective

- Identify where the search input is reflected in the HTML response.
- Determine the HTML context in which the input appears.
- Understand why HTML-encoding angle brackets alone is insufficient.
- Escape a quoted attribute and inject an event handler.
- Trigger JavaScript execution using `onmouseover`.
- Understand the appropriate defensive measures.

## 3. Step-by-Step Exploitation

### Step 1 — Open the lab

Open the PortSwigger lab and navigate to the blog search functionality.

Locate the search input field.

### Step 2 — Submit a random string

Enter a unique alphanumeric string, such as:

```
test123abc
```

Submit the search.

The random string helps identify exactly where your input appears in the resulting HTML.

### Step 3 — Intercept the request

Using Burp Suite:

1. Enable Proxy interception or locate the request in HTTP history.
2. Submit the search.
3. Send the relevant HTTP request to **Repeater**.
4. Inspect the response after sending the request.

Search for `test123abc` in the response.

### Step 4 — Identify the HTML context

Suppose the response contains an attribute similar to:

```
<input type="text" value="test123abc">
```

This is an illustrative example of a quoted attribute context. The actual vulnerable element may differ in the lab.

The important observation is that your input appears between double quotes.

The data flow is:

```
Search input
     ↓
HTTP request
     ↓
Server response
     ↓
Quoted HTML attribute
```

### Step 5 — Understand the encoding

The application HTML-encodes angle brackets.

For example:

```
<  →  &lt;
>  →  &gt;
```

This makes it harder to inject a new HTML element using `<` and `>`.

However, if the application does not also correctly encode quotation marks for the attribute context, the attribute boundary may remain vulnerable.

### Step 6 — Submit the payload

In Burp Repeater, replace the random string with:

```
"onmouseover="alert(1)
```

Send the request and inspect the response.

Conceptually, if the original attribute was:

```
<input value="USER_INPUT">
```

the resulting HTML may resemble:

```
<input value="" onmouseover="alert(1)">
```

The original attribute is closed, and a new event-handler attribute is introduced.

The exact resulting markup depends on the vulnerable element and the application's output handling.

### Step 7 — Verify the exploit in the browser

Use the lab's reflected search URL in the browser.

If necessary, right-click the affected element and inspect it to confirm whether the `onmouseover` attribute has been added.

Move the mouse pointer over the affected element.

If the injection succeeds, the browser displays an alert containing:

```
1
```

This confirms JavaScript execution.

## 4. How the Payload Works

Payload:

```
"onmouseover="alert(1)
```

**Double quote (`"`):** Terminates the original quoted attribute value.

**`onmouseover`:** Defines an event handler that runs when the pointer moves over the element.

**`alert(1)`:** JavaScript executed by the event handler.

Execution flow:

```
Reflected input
      ↓
Close existing attribute
      ↓
Inject onmouseover attribute
      ↓
User moves mouse over element
      ↓
Event handler executes
      ↓
alert(1)
```

## 5. Why Is This Vulnerable?

The root cause is insufficient context-aware output encoding.

HTML encoding is context-dependent. Encoding angle brackets is not equivalent to safely encoding every character that can affect a quoted HTML attribute.

If a double quote can terminate the original attribute, attacker-controlled input may alter the document's structure even when `<` and `>` are encoded.

**Key lesson:** A defense that only encodes angle brackets is insufficient for quoted HTML attribute contexts.

## 6. Impact

Depending on the affected application and the victim's privileges, reflected XSS may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify page content.
- Perform actions available to the victim.
- Present convincing phishing content.
- Access sensitive information exposed to JavaScript.
- Target privileged users.

The actual impact depends on the application's functionality and security controls.

## 7. Defensive Measures

### 7.1 Context-aware output encoding

Encode untrusted values for the exact context in which they are rendered.

For quoted HTML attributes, use a well-tested framework or output-encoding library that correctly encodes quotation marks and other relevant characters.

### 7.2 Use safe templating

Use a templating engine with automatic contextual output encoding. Avoid constructing HTML by concatenating untrusted strings.

### 7.3 Prefer safe DOM APIs

When displaying user input as text, use APIs such as:

```
element.textContent = userInput;
```

Avoid assigning untrusted input to `innerHTML` unless it has been appropriately sanitized for the intended use.

### 7.4 Validate input where appropriate

Validate input according to the application's expected format. Input validation complements output encoding but does not replace it.

### 7.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide an additional layer of protection. Avoid relying on CSP as the primary defense against XSS.

## 8. Remediation Summary

| Issue                                    | Recommended defense                   |
| ---------------------------------------- | ------------------------------------- |
| Input reflected into a quoted attribute  | Context-aware attribute encoding      |
| Quotation marks can alter HTML structure | Correctly encode attribute delimiters |
| User input mixed into HTML strings       | Use safe templating and DOM APIs      |
| Unexpected input accepted                | Apply appropriate input validation    |
| Additional XSS protection needed         | Implement CSP                         |

## 9. Key Takeaway

This lab demonstrates that **preventing HTML element injection does not automatically prevent attribute injection**.

The critical distinction is:

```
HTML element context ≠ HTML attribute context
```

When testing reflected XSS, identify the exact output context before choosing a payload.

Ask these questions:

1. Where is the input reflected?
2. Is it inside an HTML element, attribute, JavaScript string, or URL?
3. Which characters are encoded?
4. Can the input escape its current context?
5. What browser event can trigger the injected JavaScript?

## 10. Evidence

**Test input:**

```
test123abc
```

**Payload:**

```
"onmouseover="alert(1)
```

**Trigger:** Move the mouse pointer over the affected element.

**Expected result:** A JavaScript alert displaying `1`.

**Verification:** Confirm that the response contains an injected `onmouseover` attribute and that the event executes in the lab browser.

**Remediation:** Apply context-aware output encoding and use safe HTML-rendering practices.

## GitHub evidence checklist

### Lab documentation checklist

0 of 6

Record the random search input and intercepted request

Capture the response showing the quoted attribute context

Document the encoded angle brackets and attribute boundary

Capture the injected event handler in the rendered element

Record successful alert(1) execution

Document the remediation and retest

Suggested repository path: `Web-Pentesting/PortSwigger-Labs/Reflected-XSS/attribute-angle-brackets-encoded/README.md`

One important professional reporting point: distinguish the observed behavior from the illustrative HTML in your write-up. Include the exact vulnerable element from your actual Burp response or browser inspection, rather than claiming the illustrative `<input>` element is necessarily the lab's implementation.
