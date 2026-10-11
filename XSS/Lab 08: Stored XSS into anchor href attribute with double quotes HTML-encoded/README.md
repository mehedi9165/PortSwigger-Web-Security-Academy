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


- Comment
- Name
- Email
- Website

The **Website** field is the injection point for this lab.

### Step 2 — Submit a random test value

Enter a unique alphanumeric string into the Website field.

Example:

```text
test123abc
```

Complete the other required comment fields and submit the comment.

The purpose of this step is to determine how the application stores and renders the Website value.


<img width="1266" height="645" alt="Screenshot 2026-10-11 at 2 08 36 PM" src="https://github.com/user-attachments/assets/38a4fe2b-3e44-4ec3-8795-57425886db30" />


The anchor element creates a clickable link. The `href` attribute determines the link's destination. The important finding is that the attacker controls the destination assigned to this attribute.


The HTML portion:

```html
<a id="auther" href="https://example.com">Ronald</a>
```


<img width="972" height="489" alt="Screenshot 2026-10-11 at 2 10 49 PM" src="https://github.com/user-attachments/assets/408db2c7-790a-4c7a-949f-50054b6f2710" />



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


### Step 3 — Replace the test value with the payload

Submit the following value in the Website field:

```javascript
javascript:alert(1)
```

Complete the other required fields and submit the comment.


<img width="1264" height="747" alt="Screenshot 2026-10-11 at 2 27 32 PM" src="https://github.com/user-attachments/assets/bbbee4da-deee-4a8a-a8a6-de4e94127756" />


### Step 4 — Inspect the stored result

Reload the blog post and inspect the rendered comment.

If the injection succeeds, the anchor may resemble:

```html
<a id="auther" href="javascript:alert(1)">Ronald</a>
```

This indicates that the application has accepted a JavaScript URL as the link destination.


<img width="971" height="488" alt="Screenshot 2026-10-11 at 2 31 59 PM" src="https://github.com/user-attachments/assets/d86ac3e9-ab54-43d1-9b23-421e81c6a434" />


### Step 5 — Trigger the payload

Locate the comment associated with the test payload.

Click the **comment author's name**.

If the browser accepts and executes the JavaScript URL, an alert displaying the following value should appear:

```text
1
```

This confirms that the stored input resulted in JavaScript execution.

**Important:** The payload is triggered by clicking the author's name, not merely by displaying the comment.

---

<img width="1316" height="672" alt="Screenshot 2026-10-11 at 2 27 52 PM" src="https://github.com/user-attachments/assets/2e6938e2-5e2d-4c88-a51a-204a4642d9bf" />


---


## 4. Impact

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

## 5. Defensive Measures

### 5.1 Validate the URL Scheme

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

### 5.2 Apply Context-Aware Output Encoding

Encode untrusted values according to their output context.

Correct HTML attribute encoding is important for safely rendering values inside quoted attributes. However, encoding alone does not make an unsafe URL scheme safe.

### 5.3 Sanitize or Normalize Website Values

Parse the submitted value as a URL and validate its scheme and structure before storing or rendering it.

Where appropriate, normalize accepted URLs consistently.

### 5.4 Use Safe Rendering Practices

Use a well-tested templating framework with automatic contextual output encoding.

Avoid constructing HTML by concatenating untrusted strings.

### 5.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide additional protection against XSS. However, CSP should be treated as defense in depth, not as a substitute for URL validation and safe output handling.

