# PortSwigger Lab: Reflected XSS into a Template Literal with HTML-Sensitive Characters and Backticks Unicode-Escaped

## 1. Lab Overview

- **Vulnerability:** Reflected Cross-Site Scripting (XSS)
- **Injection point:** Search functionality
- **Injection context:** JavaScript template literal
- **Difficulty:** Practitioner
- **Security mechanisms:**
    - Angle brackets are HTML-encoded.
    - Single quotes are HTML-encoded.
    - Double quotes are HTML-encoded.
    - Backslashes and backticks are Unicode-escaped.
- **Objective:** Execute `alert(1)` inside the JavaScript template literal.

### Learning objectives

- Understand JavaScript template literals.
- Understand template literal interpolation using `${...}`.
- Identify XSS vulnerabilities inside template literals.
- Understand why escaping quotation marks and backticks does not necessarily prevent JavaScript execution.
- Use Burp Repeater to inspect reflected JavaScript input.

## . Vulnerability Description

The application's search functionality reflects user-controlled input inside a JavaScript template literal.

A template literal is a JavaScript string enclosed by backticks rather than ordinary single or double quotes.

Conceptual example:

```
const searchQuery = `USER_INPUT`;
```

The application attempts to protect this context by encoding or escaping several special characters.

However, it does not prevent JavaScript template literal interpolation.

Template literals support an expression syntax:

```
${expression}
```

Expressions placed inside `${...}` are evaluated as JavaScript.

Therefore, an attacker can inject a JavaScript expression directly into the template literal without terminating the surrounding backticks.

## 3. Tools Required

- PortSwigger Web Security Academy
- Burp Suite Proxy
- Burp Suite Repeater
- Browser

## 4. Step-by-Step Exploitation

### Step 1: Submit a normal search query

Open the lab and enter a random alphanumeric string into the search box.

For example:

```
test123
```

Submit the search request.

The purpose is to identify where the application reflects the search input.

### Step 2: Intercept the request using Burp Suite

1. Open Burp Suite.
2. Navigate to **Proxy → HTTP history**.
3. Locate the search request.
4. Right-click the request.
5. Select **Send to Repeater**.

### Step 3: Inspect the reflected input

Open the response in Burp Repeater.

Search for the random string you submitted.

You should find it reflected inside a JavaScript template literal.

Conceptual example:

```
const searchQuery = `test123`;
```

<img width="945" height="538" alt="Screenshot 2026-11-05 at 9 58 32 PM" src="https://github.com/user-attachments/assets/6ffca929-27a0-4297-8733-b5b73dd94107" />



This identifies the JavaScript template literal as the injection context.

### Step 4: Understand the filtering mechanism

The application protects several characters:

| Character | Application behavior |
| --- | --- |
| `<` | HTML-encoded |
| `>` | HTML-encoded |
| `'` | HTML-encoded |
| `"` | HTML-encoded |
| `\` | Unicode-escaped |
| Backtick (```) | Unicode-escaped |

These transformations make conventional string breakout techniques more difficult.

However, they do not prevent the use of template literal interpolation.

### Step 5: Inject the template expression

Replace your search input with:

```
${alert(1)}
```

Send the request using Burp Repeater.

The injected expression is intended to be reflected inside the existing template literal.

the resulting JavaScript becomes:

```
const searchQuery = `${alert(1)}`;
```

The browser evaluates the expression inside `${...}` when the template literal is evaluated.

### Step 6: Verify the exploit

1. Right-click the request in Burp Repeater.
2. Select **Copy URL**.
3. Paste the URL into your browser.
4. Load the page.
5. Observe the alert.


<img width="952" height="649" alt="Screenshot 2026-11-05 at 11 08 05 PM" src="https://github.com/user-attachments/assets/95d60290-bfcf-4c1a-8977-ac607a598922" />


If `alert(1)` executes, the reflected XSS vulnerability has been successfully exploited.


<img width="1043" height="530" alt="Screenshot 2026-11-05 at 11 08 38 PM" src="https://github.com/user-attachments/assets/e297feac-e524-4450-8b9e-f1064fccdf6a" />


## 5. Detailed Payload Breakdown

### Final payload

```
${alert(1)}
```

| Component | Purpose |
| --- | --- |
| `$` | Begins the template literal interpolation syntax when followed by `{`. |
| `{` | Opens the JavaScript expression block. |
| `alert(1)` | JavaScript function used to demonstrate code execution. |
| `}` | Closes the interpolation expression. |

### A. Template literal

A template literal is created using backticks:

```
const message = `Hello World`;
```

Unlike ordinary JavaScript strings, template literals support embedded expressions.

### B. Expression interpolation

The `${...}` syntax allows JavaScript expressions to execute inside a template literal.

For example:

```
const name = "Rakib";
const message = `Hello ${name}`;
```

Expected value:

```
Hello Rakib
```

The expression `${name}` is evaluated and its result is inserted into the string.

### C. Injected alert expression

In this lab:

```
${alert(1)}
```

The browser evaluates `alert(1)` as a JavaScript expression.

The alert function displays a dialog containing the number `1`.

## 6. Why Does This Work?

The application attempts to protect the template literal by escaping characters that could terminate or manipulate the surrounding string.

However, it overlooks the special interpolation syntax supported by JavaScript template literals.

The attack does not need to:

- Close the existing template literal.
- Inject a new `<script>` element.
- Use a single quote.
- Use a double quote.
- Inject a backtick.

Instead, it uses the existing template literal's expression feature.

This is why the simple payload works:

```
${alert(1)}
```

**Important:** The vulnerability is caused by inserting untrusted input into executable JavaScript without safely preventing template expression interpolation.

## 7. Security Impact

Successful reflected XSS can allow attacker-controlled JavaScript to execute in the context of the vulnerable application's origin.

Depending on the application, possible consequences include:

- Manipulation of webpage content.
- Unauthorized actions performed as the victim.
- Access to information exposed to JavaScript.
- Phishing or deceptive content presented through the trusted application.

The actual impact depends on the application's functionality and security controls.

## 8. Remediation

### Recommended defenses

**1. Avoid embedding untrusted input in executable JavaScript**

Do not concatenate user-controlled values directly into JavaScript template literals.

- **. Use safe data serialization**

Transfer server-side data using a safe serialization mechanism rather than generating JavaScript source from untrusted strings.

**3. Apply context-aware output encoding**

Encoding HTML-sensitive characters alone does not neutralize JavaScript template literal interpolation.

**4. Prefer safe DOM APIs**

Use `textContent` to display user-controlled text rather than generating executable HTML or JavaScript.

**5. Implement Content Security Policy**

A restrictive CSP provides an additional layer of protection against script injection.

##
