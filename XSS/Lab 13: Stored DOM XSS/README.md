# Stored DOM XSS — Incomplete Angle-Bracket Encoding Using `replace()`

## 1. Vulnerability

**Vulnerability:** Stored DOM-Based Cross-Site Scripting (Stored DOM XSS)

**Difficulty:** Practitioner

**Injection Point:** Blog comment functionality

**Source:** User-submitted comment

**Root Cause:** Incomplete HTML encoding using JavaScript `replace()`

**Payload:**

```html
<><img src=1 onerror=alert(1)>
```

### Description

The application contains a Stored DOM XSS vulnerability in its blog comment functionality.

To prevent cross-site scripting, the application attempts to encode angle brackets using JavaScript's `replace()` function. However, the implementation replaces only the first occurrence of the specified string.

An attacker can exploit this behavior by placing an extra pair of angle brackets at the beginning of the comment. The filter encodes those first brackets, while subsequent brackets remain unencoded.

When the application processes the stored comment and interprets the remaining input as HTML, the injected image element can trigger JavaScript execution through its `onerror` event handler.

Because the malicious comment is stored and processed when the blog post is viewed, this is a **Stored DOM XSS** vulnerability.

---

## 2. Objective

- Identify the comment field as a user-controlled input source.
- Investigate how the application processes stored comments.
- Understand the limitations of JavaScript's `replace()` function.
- Bypass incomplete angle-bracket encoding.
- Inject an HTML element into the rendered page.
- Trigger `alert(1)` to confirm JavaScript execution.
- Recommend secure alternatives for handling user-generated content.

---

## 3. Step-by-Step Exploitation

### Step 1 — Open the lab

Launch the PortSwigger lab and navigate to a blog post that supports comments.

Locate the comment submission form.

### Step 2 — Identify the comment input

Enter a harmless test comment, such as:

```
test123abc
```

Submit the comment and reload the post.

Confirm that the comment is stored and displayed on the page.

This establishes the basic input-to-output relationship.

### Step 3 — Investigate how the comment is processed

Use browser Developer Tools or Burp Suite to inspect how the application handles the comment.

Look for client-side JavaScript that processes the stored comment. The lab's filtering logic attempts to encode angle brackets using `replace()`.

The vulnerability:

```
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

### Step 4 — Submit the payload

Submit the following comment:

```html
<><img src=1 onerror=alert(1)>
```

The initial angle brackets are intended to consume the filter's first replacement, leaving the later HTML markup available for interpretation.

### Step 5 — Understand the bypass

if the application replaces only the first `<` and the first `>`, the payload can be transformed into something resembling:

```html
&lt;&gt;<img src=1 onerror=alert(1)>
```

The initial pair has been encoded, but the later `<img>` element remains intact.

The browser can therefore interpret the image markup as HTML when the application inserts the processed comment into an HTML-parsing context.

### Step 6 — Verify successful exploitation

After submitting the comment, inspect the rendered page.

If the vulnerable code processes the comment as described, the image fails to load and its `onerror` handler invokes:

```jsx
alert(1)
```

A JavaScript alert displaying `1` confirms successful execution.

Record the actual rendered DOM and alert as evidence.

---

## 

---

## 4. Why Does This Work?

The vulnerability exists because the application relies on an incomplete encoding operation.

The problematic behavior is:

```jsx
input.replace('<', '&lt;');
```

This replaces only the first occurrence of `<`.

The same limitation applies to the string-based replacement of `>`.

Consequently, an attacker can include multiple angle brackets in the comment so that some remain unencoded.

If the resulting string is subsequently interpreted as HTML, the unencoded markup may create executable elements or event handlers.

**Key lesson:** Security filters must handle every relevant occurrence and must be appropriate for the output context.

---

---

## 5. Impact

Depending on the application's functionality and the victim's privileges, Stored DOM XSS may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify page content.
- Perform actions available to the victim.
- Access information exposed to JavaScript.
- Present phishing content within a trusted interface.
- Target privileged users who view the affected comment.

The actual impact depends on the application's security controls and the privileges of the affected user.

---

## 6. Defensive Measures

### 6.1 Do not rely on single-occurrence replacement

Avoid using string-based `replace()` as a complete HTML-encoding mechanism.

A string argument replaces only the first match, so it is unsuitable for encoding every occurrence of a character unless explicitly applied repeatedly.

### 6.2 Prefer safe DOM APIs

If comments should be displayed as plain text, use:

```jsx
element.textContent = comment;
```

This treats the comment as text rather than parsing it as HTML.

### 6.3 Use a proper HTML sanitizer when HTML is required

If users are allowed to submit limited HTML, use a reputable HTML sanitization library configured with an explicit allowlist of permitted elements and attributes.

Dangerous event-handler attributes, such as `onerror`, should not be allowed in ordinary user-generated content.

### 6.4 Apply context-aware output encoding

Encode untrusted values according to the exact context in which they are rendered.

HTML text, HTML attributes, JavaScript strings, and URLs require different handling strategies.

### 6.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide an additional layer of protection against XSS. It should not replace safe DOM manipulation and correct output handling.
