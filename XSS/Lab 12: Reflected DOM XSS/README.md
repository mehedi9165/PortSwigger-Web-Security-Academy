# Reflected DOM XSS — Unsafe JSON Processing with `eval()`

## 1. Vulnerability

**Vulnerability:** Reflected DOM-Based Cross-Site Scripting (Reflected DOM XSS)

**Difficulty:** Practitioner

**Source:** User-controlled search term

**Server-side reflection:** JSON response named `search-results`

**Client-side sink:** `eval()`

**Root cause:** Incorrect escaping of backslashes and unsafe evaluation of JSON data

**Payload:**

```
\"-alert(1)}//
```

### Description

The application contains a reflected DOM XSS vulnerability in its search functionality.

When a user submits a search term, the server includes that value in a JSON response. A client-side JavaScript file, `searchResults.js`, processes the response using `eval()`.

The application escapes quotation marks but fails to escape backslashes correctly. By injecting a backslash before a quotation mark, an attacker can interfere with the intended string escaping and alter the JavaScript expression evaluated by `eval()`.

The injected expression invokes `alert(1)`, demonstrating that user-controlled data can escape its intended data context and become executable JavaScript.

This vulnerability illustrates why **JSON data must be parsed as data, not evaluated as JavaScript code**.

---

## 2. Objective

- Identify where the search term is reflected in the server's JSON response.
- Trace the reflected value into the client-side JavaScript.
- Identify `eval()` as the dangerous sink.
- Investigate how quotation marks and backslashes are escaped.
- Construct an injection that escapes the intended string context.
- Execute `alert(1)` to confirm the vulnerability.
- Recommend secure JSON handling and output-processing techniques.

---

## 3. Step-by-Step Exploitation

### Step 1 — Open the lab

Launch the PortSwigger lab and navigate to the target website.

Locate the search bar.

### Step 2 — Enable Burp Suite interception

Open Burp Suite and navigate to:

**Proxy → Intercept**

Turn interception on.

Return to the lab and search for a random test string, for example:

```
XSS
```

The browser sends a search request containing your input.

### Step 3 — Forward the search request

Return to Burp Suite.

Inspect the intercepted request and forward it so the application can process the search.

Look for the corresponding response containing the search results.

### Step 4 — Inspect the JSON response

Find the response named `search-results`.


<img width="937" height="533" alt="Screenshot 2026-10-24 at 8 43 05 PM" src="https://github.com/user-attachments/assets/e2c82dd7-f767-454a-a3e0-b3a94dd751b3" />


### Step 5 — Submit the following payload

Enter the following search term:

```
\"-alert(1)}//
```

Submit the search.



<img width="1277" height="598" alt="Screenshot 2026-10-24 at 8 59 55 PM" src="https://github.com/user-attachments/assets/3bb3df66-6868-4bc7-9520-a5debada2355" />


<img width="1273" height="596" alt="Screenshot 2026-10-24 at 8 59 30 PM" src="https://github.com/user-attachments/assets/ebc6aba5-f8c1-45f5-870d-85192cb94c25" />


The payload is designed to exploit the incorrect escaping behavior and alter the JavaScript expression constructed by the application.




### Step 6 — Verify execution

If the injection succeeds, the browser displays a JavaScript alert containing:

```
1
```

This confirms that the reflected input has resulted in JavaScript execution.



<img width="1279" height="644" alt="Screenshot 2026-10-24 at 9 43 32 PM" src="https://github.com/user-attachments/assets/2f14ae12-cb78-4c60-ad1e-910842a3b603" />


<img width="1279" height="637" alt="Screenshot 2026-10-24 at 9 43 44 PM" src="https://github.com/user-attachments/assets/dda9300d-47d9-4844-8498-c0bfb4ef63bf" />



Record the observed response and the relevant code in `searchResults.js` for your lab report.

---

## 4. Payload Analysis

The payload is:

```
\"-alert(1)}//
```

Each component contributes to changing how the resulting expression is parsed.

### `\`

The backslash is the key character in this vulnerability.

The application does not handle it correctly before placing the reflected value into the JavaScript expression.

### `"`

The double quote interacts with the application's escaping behavior. The injected backslash changes how the quote is interpreted when the response is processed.

### 

The subtraction operator separates the surrounding expression from the injected function call.

### `alert(1)`

Calls JavaScript's alert function.

### `}`

Closes the surrounding expression or object structure at the relevant point in the constructed code.

### `//`

Begins a JavaScript single-line comment. This can cause the remaining characters on that line to be ignored by the JavaScript parser.

---

## 5. Understanding the Reflected DOM XSS Data Flow

This vulnerability involves both server-side and client-side processing.

### Source

The source is the search term submitted by the user.

### Server-side reflection

The server places the search term into a JSON response.

### Client-side processing

The `searchResults.js` script receives the response and processes it.

### Dangerous sink

The response is passed to `eval()`, which parses and executes JavaScript code.

```
User-controlled search term
          ↓
HTTP request
          ↓
Server reflects the value
          ↓
JSON response
          ↓
searchResults.js
          ↓
eval()
          ↓
JavaScript execution
```

This is why the issue is classified as **Reflected DOM XSS** rather than simply a conventional server-rendered XSS vulnerability.

---

## 6. Why Does This Work?

The root cause is the combination of two weaknesses:

1. The server does not correctly handle backslashes when constructing the reflected response.
2. The client evaluates the resulting response as JavaScript using `eval()`.

The intended behavior is to treat the search term as data.

Instead, incorrect escaping allows the value to interfere with the syntax of the expression that is evaluated.

---

## 7. Impact

Depending on the application's functionality and the victim's privileges, reflected DOM XSS may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify webpage content.
- Perform actions available to the victim.
- Access information exposed to JavaScript.
- Display phishing content within the trusted application.
- Target privileged users.

The actual impact depends on the application's architecture, authentication model, browser protections, and other security controls.

---

## 8. Defensive Measures

### 8.1 Replace `eval()` with `JSON.parse()`

Do not use `eval()` to process JSON.

Unsafe pattern:

```
eval("var searchResults = " + responseText);
```

Safer pattern:

```
const searchResults = JSON.parse(responseText);
```

This parses valid JSON as data instead of evaluating the response as JavaScript code.

The application should handle parsing errors appropriately and validate the structure of the parsed result.

### 8.2 Use a JSON serializer on the server

Generate JSON using a trusted serializer instead of manually concatenating strings.

A serializer correctly handles special characters according to JSON's escaping rules.

### 8.3 Avoid Manual Escaping

Do not attempt to secure dynamically constructed JavaScript by escaping only selected characters.

The correct encoding depends on the output context, and manual escaping is prone to mistakes.

### 8.4 Treat Search Terms as Data

Keep user-controlled search terms separate from executable code.

Use ordinary variables and safe DOM APIs to display values. Do not interpolate untrusted values into JavaScript source.

### 8.5 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide additional defense in depth against XSS.

However, CSP does not replace safe JSON parsing and correct data handling.
