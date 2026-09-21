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

.....

<img width="1280" height="657" alt="Screenshot 2026-09-13 at 9 26 54 PM" src="https://github.com/user-attachments/assets/dcf37a68-2807-4072-aa1e-23fd9e223a2b" />

.....

<img width="1277" height="655" alt="Screenshot 2026-09-13 at 9 27 22 PM" src="https://github.com/user-attachments/assets/3ec76300-8195-4dee-9ee8-9286a80c4761" />

......

<img width="1255" height="656" alt="Screenshot 2026-09-13 at 9 29 46 PM" src="https://github.com/user-attachments/assets/2fc8769c-45a0-4f1e-bc2c-f9e4e6408e5e" />
....



### Step 3 — Inject XSS Payload

Replace the test input with:

```html
<script>alert(1)</script>
```



......

<img width="1279" height="651" alt="Screenshot 2026-09-13 at 9 30 58 PM" src="https://github.com/user-attachments/assets/bf2e01c4-8605-4b07-a75e-62e5251ef168" />

....



### Step 4 — Trigger the Payload

Return to or reload the blog post.

The stored comment is retrieved from the server and rendered in the page.

A JavaScript alert displaying:

```
1
```

should appear.

...


<img width="1277" height="656" alt="Screenshot 2026-09-13 at 9 31 15 PM" src="https://github.com/user-attachments/assets/edb61ddc-4f09-4d55-90fd-f3ae73b53b6c" />


.....


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

