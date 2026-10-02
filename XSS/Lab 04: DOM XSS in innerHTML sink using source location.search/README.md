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



......
<img width="1420" height="727" alt="Screenshot 2026-10-02 at 11 38 33 AM" src="https://github.com/user-attachments/assets/5bde202e-ab8e-4b9c-aadd-a86ffe6af990" />


.......




## Step 3 — Submit the XSS Payload

Enter the following payload:

```html
<img src=1 onerror=alert(1)>
```

Then click:

**Search**

---

......

<img width="1383" height="758" alt="Screenshot 2026-10-02 at 11 47 00 AM" src="https://github.com/user-attachments/assets/2bb7bd2b-b0df-4a90-809a-3c4e8aa1f9af" />


......




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



......




---

# 4. Confirm Successful Exploitation

If the vulnerability is successfully exploited, the browser displays a JavaScript alert containing:

```
1
```


.....



<img width="1351" height="631" alt="Screenshot 2026-10-02 at 11 49 18 AM" src="https://github.com/user-attachments/assets/88f03fa7-52c1-411f-9858-6c7a5b79df0f" />


......

The appearance of the alert confirms that JavaScript execution occurred in the application's security context.

---

.....




<img width="1432" height="660" alt="Screenshot 2026-10-02 at 11 50 44 AM" src="https://github.com/user-attachments/assets/f0bffe6d-c08a-458e-a15c-788691a9c584" />


......


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



---

# 6. Impact

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

# 7. Defensive Measures

## 7.1 Context-Aware Output Encoding

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

## 7.2 HTML Sanitization

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

## 7.3 Avoid Dangerous DOM APIs

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

## 7.4 Content Security Policy

Implement a strong **Content Security Policy (CSP)** to provide an additional layer of protection against XSS.

For example:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be considered a defense-in-depth measure rather than a replacement for proper output encoding.

---

## 7.5 Secure Cookie Configuration

For authenticated applications, cookies should use appropriate security attributes such as:

```
HttpOnly
Secure
SameSite
```

`HttpOnly` can help prevent JavaScript from directly reading session cookies.

---



