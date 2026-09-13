# 🎯 Objective

Exploit a **Stored Cross-Site Scripting (XSS)** vulnerability in the blog's comment functionality.

The goal is to submit a comment that executes JavaScript when the blog post is viewed.

---

## 🔍 Methodology

### Step 1 — Locate the Comment Function

Open the vulnerable blog post and scroll to the **comment section**.

The application allows users to submit comments containing their name, email, and website.

### Step 2 — Test for HTML Injection

In the **Comment** field, enter:

```html
<h1>Test</h1>
```

Enter valid values for the other required fields and submit the comment.

If the browser renders the heading as HTML, the application is accepting HTML without proper encoding.

### Step 3 — Inject XSS Payload

Replace the test input with:

```html
<script>alert(1)</script>
```

Fill in the required:

- **Name**
- **Email**
- **Website**

Then click **Post comment**.

### Step 4 — Trigger the Payload

Return to or reload the blog post.

The stored comment is retrieved from the server and rendered in the page.

A JavaScript alert displaying:

```
1
```

should appear.

### Step 5 — Confirm the Vulnerability

The alert confirms that the JavaScript payload was:

```
Submitted → Stored by the application → Retrieved → Executed by the browser
```

---

## 💥 Payload

```html
<script>alert(1)</script>
```

## 🧠 Why It Works

Unlike reflected XSS, the malicious input is **stored on the server** as part of the comment.

When another user—or the attacker—views the affected blog post, the stored payload is inserted into the HTML without proper encoding, causing the browser to execute the JavaScript.

```
Attacker
   ↓
Comment Submission
   ↓
Server / Database
   ↓
Stored Malicious Input
   ↓
Blog Post Viewed
   ↓
HTML Response
   ↓
JavaScript Execution
```

## 🔑 Key Takeaway

**Stored XSS occurs when malicious user input is persistently stored by an application and later rendered in a web page without appropriate output encoding or sanitization.**

