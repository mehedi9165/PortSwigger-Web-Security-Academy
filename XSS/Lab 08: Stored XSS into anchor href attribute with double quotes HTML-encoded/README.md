# Stored XSS into Anchor `href` Attribute with Double Quotes HTML-Encoded

## 1. Vulnerability

**Vulnerability:** Stored Cross-Site Scripting (Stored XSS)

**Difficulty:** Apprentice

**Injection Point:** Comment functionality — Website field

**Source:** User-submitted Website value

**Sink:** Anchor (`href`) attribute

**Payload:**

```javascript
javascript:alert(1)
```

### Description

The application contains a stored XSS vulnerability in its comment functionality.

When submitting a comment, the user can provide a website URL. The application stores this value and later inserts it into the `href` attribute of the comment author's name.

Although the application HTML-encodes double quotes, it does not adequately validate the URL scheme. Consequently, an attacker can submit a `javascript:` URL instead of a legitimate website address.

When another user clicks the comment author's name, the browser executes the JavaScript URL, triggering `alert(1)`.

Because the malicious value is stored and rendered when the comment is viewed, this is a **Stored XSS** vulnerability.

---

## 2. Objective

- Identify how the comment Website field is processed.
- Determine where the supplied value appears in the rendered HTML.
- Understand the security implications of a user-controlled anchor `href`.
- Inject a `javascript:` URL into the anchor attribute.
- Trigger JavaScript execution when the comment author's name is clicked.
- Identify suitable remediation measures.

---

## 3. Step-by-Step Exploitation

### Step 1 — Open the lab

Launch the PortSwigger lab and navigate to a blog post that allows comments.

Locate the comment form. It typically contains fields such as:

- Name
- Email
- Website
- Comment

The **Website** field is the injection point for this lab.

### Step 2 — Submit a random test value

Enter a unique alphanumeric string into the Website field.

Example:

```text
test123abc
```

Complete the other required comment fields and submit the comment.

The purpose of this step is to determine how the application stores and renders the Website value.

### Step 3 — Intercept the comment submission

Using Burp Suite:

1. Submit the comment through the browser.
2. Locate the comment submission request in Proxy HTTP history.
3. Send the request to **Repeater**.
4. Inspect the request parameters and response.

Identify the parameter corresponding to the Website field.

The parameter name may vary, so use the actual request rather than assuming a particular name.

### Step 4 — Retrieve the rendered comment

Make a second request in the browser to reload the blog post containing your comment.

Intercept this request and send it to a separate Repeater tab.

Inspect the response and search for:

```text
test123abc
```

You should find that the submitted value has been inserted into an anchor's `href` attribute.

For example, the rendered HTML may resemble:

```html
<a href="https://test123abc">Comment Author</a>
```

This is an illustrative example. Record the actual HTML structure observed in your lab response.

The data flow is:

```text
Website field
      ↓
Comment submission
      ↓
Application stores the value
      ↓
Blog post is requested again
      ↓
Stored value is rendered in href
```

### Step 5 — Understand the HTML context

The value appears inside an anchor attribute:

```html
<a href="USER_CONTROLLED_VALUE">Comment Author</a>
```

The anchor element creates a clickable link.

The `href` attribute determines the link's destination.

The important finding is that the attacker controls the destination assigned to this attribute.

### Step 6 — Replace the test value with the payload

Submit another comment, this time using the following value in the Website field:

```javascript
javascript:alert(1)
```

Complete the other required fields and submit the comment.

The application should store the supplied value and use it as the destination of the comment author's name.

### Step 7 — Inspect the stored result

Reload the blog post and inspect the rendered comment.

If the injection succeeds, the anchor may resemble:

```html
<a href="javascript:alert(1)">Comment Author</a>
```

This indicates that the application has accepted a JavaScript URL as the link destination.

### Step 8 — Trigger the payload

Locate the comment associated with the test payload.

Click the **comment author's name**.

If the browser accepts and executes the JavaScript URL, an alert displaying the following value should appear:

```text
1
```

This confirms that the stored input resulted in JavaScript execution.

**Important:** The payload is triggered by clicking the author's name, not merely by displaying the comment.

---

## 4. How the Payload Works

The payload is:

```javascript
javascript:alert(1)
```

It consists of two components.

### `javascript:`

This is a URL scheme that allows JavaScript code to be executed when used in a suitable browser context, such as a clickable anchor destination.

### `alert(1)`

This is the JavaScript expression executed when the link is activated.

The execution flow is:

```text
User submits comment
        ↓
Website field contains payload
        ↓
Application stores the comment
        ↓
Payload is rendered in anchor href
        ↓
Another user clicks author name
        ↓
Browser executes JavaScript URL
        ↓
alert(1)
```

---

## 5. Why Does This Work?

The application encodes double quotes in HTML output but does not adequately restrict the URL scheme.

These are separate security concerns.

**HTML attribute encoding** helps prevent an attacker from breaking out of a quoted attribute.

**URL scheme validation** ensures that a link uses an allowed scheme, such as `https:` or an appropriate relative URL.

Encoding quotation marks does not make a `javascript:` URL safe.

The vulnerable behavior is conceptually:

```html
<a href="USER_CONTROLLED_URL">Comment Author</a>
```

The browser interprets the `href` value as a navigation destination. When that value uses an executable JavaScript URL, clicking the link may execute the supplied code.

---

## 6. Why Is This Stored XSS?

The malicious value is saved as part of a comment and appears again when the blog post is loaded.

This distinguishes Stored XSS from Reflected XSS.

**Stored XSS:**

```text
Attacker submits comment
        ↓
Application stores payload
        ↓
Another user opens the post
        ↓
Stored payload is rendered
        ↓
User clicks the author's name
        ↓
JavaScript executes
```

**Reflected XSS:**

```text
Attacker sends crafted request
        ↓
Server reflects input in response
        ↓
Victim opens crafted URL/request
        ↓
JavaScript executes
```

In this lab, the persistent comment storage makes the attack Stored XSS.

---

## 7. Impact

Depending on the application and the victim's privileges, Stored XSS may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify page content.
- Perform actions available to the victim.
- Present phishing content within a trusted website.
- Access information exposed to JavaScript.
- Target administrators or other privileged users.

The actual impact depends on the application's functionality, browser protections, and other security controls.

The fact that the payload requires a click does not eliminate the vulnerability; it means the demonstrated exploit requires user interaction.

---

## 8. Defensive Measures

### 8.1 Validate the URL Scheme

The application should allow only appropriate URL schemes for the Website field.

For a field intended to contain external websites, an allowlist might permit:

```text
https:
http:
```

The application should reject schemes such as:

```text
javascript:
```

Validation should use a proper URL parser and enforce the intended policy rather than relying only on string matching.

### 8.2 Apply Context-Aware Output Encoding

Encode untrusted values according to their output context.

Correct HTML attribute encoding is important for safely rendering values inside quoted attributes. However, encoding alone does not make an unsafe URL scheme safe.

### 8.3 Sanitize or Normalize Website Values

Parse the submitted value as a URL and validate its scheme and structure before storing or rendering it.

Where appropriate, normalize accepted URLs consistently.

### 8.4 Use Safe Rendering Practices

Use a well-tested templating framework with automatic contextual output encoding.

Avoid constructing HTML by concatenating untrusted strings.

### 8.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide additional protection against XSS. However, CSP should be treated as defense in depth, not as a substitute for URL validation and safe output handling.

---

## 9. Remediation Summary

| Issue | Recommended Defense |
|---|---|
| User controls the anchor destination | Validate and allowlist URL schemes |
| `javascript:` URL is accepted | Reject unsafe schemes |
| User data is rendered in `href` | Apply context-aware output encoding |
| URL values are trusted without parsing | Use a proper URL parser and validation policy |
| Stored content can execute JavaScript | Secure rendering and input handling |
| Additional XSS protection is needed | Implement CSP |

---

## 10. Key Takeaway

This lab demonstrates that **HTML attribute encoding and URL validation solve different security problems**.

The critical data flow is:

```text
Website input
      ↓
Stored comment
      ↓
Anchor href
      ↓
javascript: URL
      ↓
User clicks author name
      ↓
JavaScript execution
```

When testing XSS in HTML attributes, ask:

1. Where does the input appear in the rendered HTML?
2. Is it inside a quoted attribute?
3. Is the attribute a URL destination?
4. Which URL schemes does the application permit?
5. Does JavaScript execute when the link is activated?

---

## 11. Evidence

**Test value:**

```text
test123abc
```

**Observed context:**

```html
<a href="USER_CONTROLLED_VALUE">Comment Author</a>
```

**Payload:**

```javascript
javascript:alert(1)
```

**Trigger:** Click the comment author's name.

**Expected result:** A JavaScript alert displaying `1`.

**Verification:** Confirm that the submitted value persists in the comment and that clicking the author name triggers JavaScript execution.

**Remediation:** Validate URL schemes and apply context-aware output encoding.

**Lab status:** Mark as solved only after confirming successful execution in the lab environment.
