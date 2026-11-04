# PortSwigger Lab: Reflected XSS in Canonical Link Tag

## 1. Lab Overview

- **Vulnerability:** Reflected Cross-Site Scripting (XSS)
- **Injection point:** URL query parameter
- **HTML context:** Canonical link tag
- **Difficulty:** Practitioner
- **Security control:** Angle brackets are HTML-encoded
- **Browser requirement:** Chrome
- **Objective:** Inject an HTML attribute that calls `alert(1)` when the user presses a specified keyboard shortcut.

### Learning objectives

- Understand canonical link tags.
- Identify attribute injection vulnerabilities.
- Understand how `accesskey` works.
- Use an existing HTML element to execute JavaScript without injecting a new tag.
- Understand the relationship between `accesskey` and `onclick`.

## . Vulnerability Description

The application reflects user-controlled input into a canonical link tag on the home page.

A canonical link tag normally identifies the preferred URL for a webpage. For example:

```
<link rel="canonical" href="https://example.com/page">
```

In this lab, user input is reflected into the canonical link tag. Although the application encodes angle brackets (`<` and `>`), it does not adequately protect the HTML attribute context.

This creates an opportunity to inject additional attributes into the existing link element.

**Important:** HTML-encoding angle brackets does not prevent attribute injection when an attacker can escape an existing quoted attribute.

## 3. Exploit Payload

The lab solution uses this URL:

```
https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```

Replace `YOUR-LAB-ID` with your actual lab identifier.

The injected query parameter is:

```
%27accesskey=%27x%27onclick=%27alert(1)
```

After URL decoding:

```
'accesskey='x'onclick='alert(1)
```

The resulting HTML attributes are intended to resemble:

```
accesskey="x" onclick="alert(1)"
```

The exact surrounding HTML depends on the application's canonical link markup.

## 4. Step-by-Step Exploitation

### Step 1: Open the lab

Launch the PortSwigger lab and open its home page.

### Step 2: Understand the reflection point

The application reflects the query parameter into the canonical link tag.

The relevant HTML structure is conceptually similar to:

```
<link rel="canonical" href="USER-CONTROLLED-VALUE">
```

The important observation is that user input appears inside a quoted HTML attribute.

### Step 3: Inject the payload

Visit the following URL, replacing the lab identifier:

```
https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```

The payload attempts to close the existing attribute and introduce two new attributes:

- `accesskey`
- `onclick`


<img width="1151" height="640" alt="Screenshot 2026-11-04 at 2 21 02 PM" src="https://github.com/user-attachments/assets/42f4a6fd-1254-42c0-bc65-53b1cc8bb520" />



### Step 4: Trigger the keyboard shortcut

The injected element has the access key `x`.

The lab specifies that the simulated user will press one of the following combinations:

| Operating system | Keyboard shortcut |
| --- | --- |
| Windows | ALT + SHIFT + X |
| macOS | CTRL + ALT + X |
| Linux | ALT + X |

When the browser activates the access key, the element receives the relevant interaction and its `onclick` handler executes.

The expected result is:

```
alert(1)
```

### 

## 5. Why Does This Work?

The application protects against angle-bracket injection but does not properly encode user input for its HTML attribute context.

The attacker does not need to inject a new element such as:

```
<script>alert(1)</script>
```

Instead, the attacker modifies an existing element by injecting new attributes.

The `accesskey` attribute provides a way for the simulated victim to activate the element using a keyboard shortcut. The `onclick` handler then executes JavaScript.

This demonstrates why HTML attribute injection remains dangerous even when angle brackets are encoded.

## 6. Why Is Chrome Required?

The lab explicitly specifies Chrome because access-key behavior and keyboard shortcut handling can vary between browsers and operating systems.

The exact key combination depends on the browser and operating system.

For this lab, follow the specified combinations rather than assuming that the same shortcut works universally.

## 7. Security Impact

If an attacker can inject executable event-handler attributes into a page, reflected XSS may allow JavaScript to execute in the context of the vulnerable application's origin.

Potential consequences include:

- Manipulation of webpage content.
- Unauthorized actions performed as the victim.
- Access to information exposed to JavaScript.
- Phishing or deceptive content presented within the trusted website.

The actual impact depends on the application's security controls and functionality.

## 8. Remediation

### Recommended defenses

1. **Context-aware output encoding:** Encode user-controlled values specifically for HTML attribute contexts.
2. **Safe attribute handling:** Avoid concatenating untrusted input into HTML attributes.
3. **Safe DOM APIs:** Use appropriate DOM properties and APIs instead of constructing HTML strings.
4. **Content Security Policy:** Use CSP to restrict script execution as an additional security layer.
5. **Avoid inline event handlers:** Prefer JavaScript event listeners registered in trusted application code.

### Key defensive lesson

Encoding `<` and `>` alone is insufficient. Applications must protect against attribute breakout and ensure that untrusted input cannot introduce executable event handlers.
