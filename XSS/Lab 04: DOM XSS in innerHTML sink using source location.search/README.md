# Stored Cross-Site Scripting (XSS) — `img` `onerror` Event Handler

## 1. Vulnerability

**Vulnerability:** Stored Cross-Site Scripting (Stored XSS)

**Severity:** Medium

**Attack Vector:** Malicious JavaScript injected through a user-controlled search/post input

**Payload:**

```html
<img src=1 onerror=alert(1)>
```

### Description

The application accepts user-controlled HTML input and renders it in the page without properly encoding or sanitizing the supplied content.

The payload uses an `<img>` element with an invalid `src` value. When the browser attempts to load the image and the request fails, the `onerror` event is triggered and executes the JavaScript supplied in the event handler.

This demonstrates that attacker-controlled HTML and JavaScript can be executed in the victim's browser.

---

# 2. Objective

The objective of this lab is to:

- Identify a location where user-controlled input is reflected or stored.
- Determine whether HTML markup is interpreted by the application.
- Test whether JavaScript can be executed through an HTML event handler.
- Demonstrate a Stored XSS vulnerability using an `onerror` event.
- Understand the potential security impact of insufficient output encoding and input sanitization.

---

# 3. Step-by-Step Exploitation

## Step 1 — Identify the Input Field

Navigate to the vulnerable application's page.

Locate the **Search** input field.

Example:

```
+------------------------------+
| Search: [                  ] |
|              [ Search ]      |
+------------------------------+
```

The search field accepts user-controlled input.

---

## Step 2 — Test HTML Injection

Enter a simple HTML element into the search field:

```html
<b>Test</b>
```

Click:

**Search**

If the word `Test` appears in bold, this indicates that the application is interpreting the supplied input as HTML rather than displaying it purely as text.

This is an important indicator that further XSS testing may be possible.

---

## Step 3 — Submit the XSS Payload

Enter the following payload:

```html
<img src=1 onerror=alert(1)>
```

Then click:

**Search**

---

## Step 4 — Understand the Payload

The payload consists of three important components:

```html
<img src=1 onerror=alert(1)>
```

### `<img>`

Creates an image element.

### `src=1`

The browser attempts to load an image from the value `1`.

Because this is not a valid image resource, the image fails to load.

### `onerror`

The `onerror` event executes when the image fails to load.

### `alert(1)`

The JavaScript executed by the event handler.

Therefore, the execution flow is:

```
User-controlled input
        ↓
<img> element is rendered
        ↓
Browser attempts to load src=1
        ↓
Image loading fails
        ↓
onerror event fires
        ↓
alert(1) executes
```

---

# 4. Confirm Successful Exploitation

If the vulnerability is successfully exploited, the browser displays a JavaScript alert containing:

```
1
```

Example:

```
+----------------------+
|          1           |
|                     |
|        [ OK ]       |
+----------------------+
```

The appearance of the alert confirms that JavaScript execution occurred in the application's security context.

---

# 5. Why Does This Work?

The application is allowing attacker-controlled HTML to reach the browser without appropriate output encoding or sanitization.

Instead of safely displaying:

```
<img src=1 onerror=alert(1)>
```

as text, the browser interprets it as HTML:

```html
<img src=1 onerror=alert(1)>
```

The browser then processes the `img` element and triggers the `onerror` handler when the image fails to load.

---

# 6. Vulnerability Flow

```
Attacker
   │
   │ Injects malicious HTML
   ▼
Application Input
   │
   │ Insufficient sanitization
   ▼
Application stores/renders input
   │
   ▼
Victim's Browser
   │
   │ Interprets injected HTML
   ▼
<img src=1 onerror=alert(1)>
   │
   ▼
Image fails to load
   │
   ▼
onerror event executes
   │
   ▼
JavaScript execution
```

---

# 7. Impact

Depending on the application's security controls and the privileges of the affected user, XSS can potentially allow an attacker to:

- Execute arbitrary JavaScript in another user's browser.
- Perform actions on behalf of the victim.
- Modify webpage content.
- Access sensitive information available to JavaScript.
- Conduct phishing attacks within the trusted application interface.
- Abuse authenticated application functionality.
- Target privileged users such as administrators.

The actual impact depends on the application's architecture, authentication model, browser protections, and available security controls.

---

# 8. Defensive Measures

## 8.1 Context-Aware Output Encoding

User-controlled data should be properly encoded before being inserted into HTML.

For example, characters such as:

```
<
>
"
'
&
```

should be appropriately encoded when they are expected to be displayed as text.

---

## 8.2 HTML Sanitization

If the application legitimately needs to support HTML input, use a well-maintained HTML sanitization library with an allowlist approach.

Only explicitly permitted HTML elements and attributes should be allowed.

For example, potentially dangerous attributes such as:

```html
onerror
onclick
onload
```

should generally not be permitted in user-generated content.

---

## 8.3 Avoid Dangerous DOM APIs

Developers should avoid inserting untrusted data directly into dangerous DOM sinks such as:

```jsx
innerHTML
document.write()
```

Prefer safer alternatives such as:

```jsx
textContent
```

when HTML rendering is not required.

---

## 8.4 Content Security Policy

Implement a strong **Content Security Policy (CSP)** to provide an additional layer of protection against XSS.

For example:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be considered a defense-in-depth measure rather than a replacement for proper output encoding.

---

## 8.5 Secure Cookie Configuration

For authenticated applications, cookies should use appropriate security attributes such as:

```
HttpOnly
Secure
SameSite
```

`HttpOnly` can help prevent JavaScript from directly reading session cookies.

---

# 9. Remediation Summary

| Issue | Recommended Defense |
| --- | --- |
| Untrusted HTML rendered directly | Context-aware output encoding |
| Dangerous HTML elements accepted | HTML allowlist/sanitization |
| Event-handler attributes accepted | Remove/deny `on*` attributes |
| Unsafe DOM manipulation | Prefer `textContent` |
| XSS defense in depth | Implement CSP |
| Session-cookie exposure | Use `HttpOnly`, `Secure`, `SameSite` |

---

# 10. Key Takeaway

This lab demonstrates how an attacker can exploit insufficient input handling by injecting an HTML element containing an event handler.

The important concept is:

```
Untrusted Input
      ↓
HTML Injection
      ↓
Event Handler
      ↓
JavaScript Execution
      ↓
Stored/Reflected XSS
```

The primary defense is **context-appropriate output encoding**, supported by proper HTML sanitization and security controls such as CSP.

---

## Evidence

**Payload used:**

```html
<img src=1 onerror=alert(1)>
```

**Expected result:**

```
JavaScript alert displaying: 1
```

**Vulnerability confirmed:** Yes — successful JavaScript execution demonstrates XSS.

For your GitHub portfolio, I’d also add **2–3 screenshots** under the relevant sections: **payload entered → application response → `alert(1)` execution**. That makes the lab much more credible as a practical pentesting record.
