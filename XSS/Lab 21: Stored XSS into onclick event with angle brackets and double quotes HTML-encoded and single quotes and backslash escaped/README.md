# PortSwigger Lab: Stored XSS into Onclick Event with Angle Brackets and Double Quotes HTML-Encoded and Single Quotes and Backslash Escaped

## 1. Lab Overview

- **Vulnerability:** Stored Cross-Site Scripting (Stored XSS)
- **Injection point:** Comment functionality — Website field
- **Injection context:** HTML attribute containing a JavaScript `onclick` event handler
- **Difficulty:** Practitioner
- **Security mechanisms:**
    - Angle brackets (`<` and `>`) are HTML-encoded.
    - Double quotes (`"`) are HTML-encoded.
    - Single quotes (`'`) are escaped.
    - Backslashes (`\`) are escaped.
- **Objective:** Submit a comment that executes `alert(1)` when the comment author's name is clicked.

### Learning objectives

- Understand stored XSS vulnerabilities.
- Identify JavaScript injection inside HTML event-handler attributes.
- Understand HTML character references such as `&apos;`.
- Learn how HTML parsing interacts with JavaScript event-handler execution.
- Use Burp Suite to trace input from submission to rendered output.

## 2. Vulnerability Description

The application provides a comment functionality that includes a Website field.

The submitted website value is stored by the application and later reflected into an `onclick` event handler associated with the comment author's name.

The resulting HTML is  similar to:

```
<a
    id="author"
    href="https://example.com"
    onclick="var tracker={track(){}};tracker.track('https://example.com');"
>
    Robert
</a>
```

The application attempts to protect the input by encoding or escaping several special characters.

However, the filtering mechanism does not correctly handle HTML character references in the context of the event-handler attribute.

This creates an opportunity to manipulate the JavaScript expression executed when the author's name is clicked.

Because the malicious input is stored and executed when the comment is viewed, this is a **Stored XSS vulnerability**.

## 3. Tools Required

- PortSwigger Web Security Academy
- Burp Suite Proxy
- Burp Suite Repeater
- Browser Developer Tools

## 4. Step-by-Step Exploitation

### Step 1: Submit a normal comment

1. Open the lab.
2. Navigate to a blog post containing the comment functionality.
3. Enter a normal comment.
4. Enter a random alphanumeric string into the Website field.

Example:

```
random123
```

1. Submit the comment.

The purpose is to identify how the application stores and processes the Website field.

### Step 2: Intercept the comment submission request

1. Open Burp Suite.
2. Navigate to **Proxy → HTTP history**.
3. Locate the comment submission request.
4. Right-click the request.
5. Select **Send to Repeater**.

Inspect the request and identify the parameter containing the Website value.

### Step 3: Inspect the rendered comment

Make a second request in the browser to view the blog post containing your comment.

Intercept this request using Burp Suite and send it to another Repeater tab.

Inspect the response and search for your random alphanumeric string.

You should find that the Website value has been reflected inside an `onclick` event-handler attribute associated with the comment author's name.

Conceptual example:

```
<a onclick="someFunction('random123')">
    Comment Author
</a>
```

This identifies the injection context as JavaScript embedded inside an HTML attribute.

### Step 4: Construct the malicious Website value

Replace the normal Website value with:

```
https://example.com?&apos;-alert(1)-&apos;
```

Submit the comment.

The important part of this payload is:

```
&apos;-alert(1)-&apos;
```

The `&apos;` character reference represents a single quote in HTML.

When the browser parses the HTML attribute, the character reference is decoded into a quote character.

This can change the structure of the JavaScript expression inside the event handler.

### Step 5: Verify the stored payload

After submitting the comment:

1. Open the blog post again.
2. Inspect the rendered comment.
3. Confirm that the comment author name is associated with the injected `onclick` handler.
4. Right-click the relevant request in Burp Repeater.
5. Select **Copy URL**.
6. Open the copied URL in your browser.

### Step 6: Trigger the stored XSS

Click the comment author's name.

The browser activates the `onclick` event handler and should execute:

```
alert(1)
```

If the alert appears, the lab has been successfully solved.

**Important:** Unlike reflected XSS, this payload is stored in the application's comment data. It executes when the affected comment is viewed and the author name is clicked.

## 5. Detailed Payload Breakdown

### Final payload

```
https://example.com?&apos;-alert(1)-&apos;
```

| Component | Purpose |
| --- | --- |
| https://example.com? | Provides a URL-like value for the Website field. |
| `&apos;` | HTML character reference representing a single quote. |
| `-` | Separates the original JavaScript expression from the injected expression. |
| `alert(1)` | JavaScript execution proof. |
| `-` | Helps complete the surrounding JavaScript expression. |
| `&apos;` | Represents another single quote to balance the expression. |

### 7. Why Does This Work?

The vulnerability results from a mismatch between HTML parsing and JavaScript parsing.

The application attempts to escape special characters before inserting the value into the event-handler attribute.

However, the browser also processes HTML character references when parsing HTML attributes.

The injected `&apos;` sequences are interpreted as quote characters, potentially changing the JavaScript expression that the event handler executes.

This demonstrates that escaping literal single quotes alone is not a sufficient defense when the application inserts untrusted input into an HTML attribute containing JavaScript.

## 8. Root Cause

The root cause is unsafe insertion of user-controlled data into an executable JavaScript event-handler attribute.

The application relies on incomplete escaping rather than safely handling the HTML and JavaScript parsing contexts.

Additionally, using inline event handlers makes this type of vulnerability more difficult to prevent reliably.

## 9. Security Impact

Stored XSS can be especially dangerous because malicious input persists in the application and may affect multiple users who view the same content.

Depending on the application's functionality, successful exploitation may allow:

- Manipulation of webpage content.
- Unauthorized actions performed as a victim.
- Access to information exposed to JavaScript.
- Phishing or deceptive content displayed through a trusted website.

The actual impact depends on the application's permissions and security controls.

## 10. Remediation

### Recommended defenses

**1. Avoid inline JavaScript event handlers**

Instead of constructing `onclick` attributes dynamically, register event listeners in trusted application code.

**2. Apply context-aware output encoding**

HTML attribute encoding and JavaScript string encoding are different requirements. Protect each context appropriately.

**3. Use safe DOM APIs**

Use DOM properties and event listeners rather than constructing HTML strings containing untrusted input.

**4. Validate and sanitize Website values**

Validate URLs using an appropriate URL parser and an allowlist of permitted schemes, such as HTTPS.

**5. Implement Content Security Policy**

A restrictive CSP can reduce the impact of inline script injection, although it should not replace safe output handling.

##
