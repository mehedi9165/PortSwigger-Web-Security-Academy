# Lab: Reflected XSS into HTML Context with Nothing Encoded

## 🎯 Objective

Exploit a reflected XSS vulnerability where user input is reflected into HTML without encoding.

## 🔍 Steps

### 1. Identify Reflection

Enter:

```text
test
```

Click **Search** and observe that the input is reflected in the response.


<img width="1435" height="883" alt="Screenshot 2026-09-09 at 9 56 28 PM" src="https://github.com/user-attachments/assets/3285e631-ee5d-4678-8df4-d8080403676d" />


### 2. Test HTML Injection

Enter:

```html
<h1>TEST</h1>
```

If the browser renders it as a heading, HTML injection is possible.

<img width="1463" height="874" alt="Screenshot 2026-09-09 at 9 57 29 PM" src="https://github.com/user-attachments/assets/a2526efe-1365-4304-8a8a-065bd0c2da4c" />

<img width="1461" height="875" alt="Screenshot 2026-09-09 at 9 58 07 PM" src="https://github.com/user-attachments/assets/a079aa85-dabc-441a-8148-9a48d033235c" />



### 3. Execute XSS

Enter:

```html
<script>alert(1)</script>
```

Click **Search**.

### 4. Result

The JavaScript executes and an alert displaying `1` appears.


<img width="1462" height="873" alt="Screenshot 2026-09-09 at 9 58 59 PM" src="https://github.com/user-attachments/assets/4e042d7f-7090-4204-a0be-67028b873cf1" />


## 🧠 Why It Works

The application reflects attacker-controlled input directly into the HTML response without encoding:

```text
User Input → Server → HTML Response → Browser → JavaScript Execution
```

## 💥 Payload

```html
<script>alert(1)</script>
```

## 🔑 Key Takeaway

**Reflected XSS occurs when untrusted input is reflected into a page and interpreted as executable HTML/JavaScript.**

