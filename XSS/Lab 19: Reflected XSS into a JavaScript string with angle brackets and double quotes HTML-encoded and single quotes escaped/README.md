## 1. Lab Overview

- **Vulnerability:** Reflected Cross-Site Scripting (XSS)
- **Injection point:** Search functionality
- **Injection context:** JavaScript string inside an HTML script block
- **Difficulty:** Practitioner
- **Security mechanisms:**
    - Angle brackets (`<` and `>`) are HTML-encoded.
    - Double quotes (`"`) are HTML-encoded.
    - Single quotes (`'`) are backslash-escaped.
    - Backslashes () are not escaped.
- **Objective:** Bypass the escaping mechanism, break out of the JavaScript string, and execute `alert(1)`.

### Learning objectives

- Understand JavaScript string escaping.
- Identify incomplete input sanitization.
- Understand how backslashes interact with escaped quotes.
- Learn how to escape a JavaScript string without using HTML tags.
- Use Burp Repeater to test input transformations.

## . Vulnerability Description

The application's search functionality reflects user-controlled input into a JavaScript string used for search query tracking.

The vulnerable implementation is conceptually similar to:

```
<script>
    var searchQuery = 'USER_INPUT';
</script>
```

The application attempts to protect this context using several transformations:

| Character | Application behavior |
| --- | --- |
| `<` | HTML-encoded |
| `>` | HTML-encoded |
| `"` | HTML-encoded |
| `'` | Backslash-escaped |
| `\` | Not escaped |

Although the application protects several characters, it fails to escape backslashes.

This is important because a backslash changes how the JavaScript parser interprets the character that follows it.

An attacker can exploit this behavior to neutralize the application's single-quote escaping and escape from the JavaScript string.

## 3. Tools Required

- PortSwigger Web Security Academy
- Burp Suite Proxy
- Burp Suite Repeater
- Browser

## 4. Step-by-Step Exploitation

### Step 1: Submit a normal search query

Open the lab and submit a random alphanumeric search term.

Example:

```
test123
```

The purpose is to identify how the application reflects user-controlled input.

### Step 2: Intercept the request

1. Open Burp Suite.
2. Navigate to **Proxy → HTTP history**.
3. Locate the search request.
4. Right-click the request.
5. Select **Send to Repeater**.




### Step 3: Inspect the reflected input

Open the response in Burp Repeater.

Search for the random string. You should find it reflected inside a JavaScript string.


```
var searchQuery = 'test123';
```

This identifies the JavaScript string as the injection context.


<img width="800" height="478" alt="Screenshot 2026-11-05 at 7 50 50 PM" src="https://github.com/user-attachments/assets/93321091-60be-42b7-be39-382eb4eaf5b1" />



<img width="943" height="678" alt="Screenshot 2026-11-05 at 7 56 38 PM" src="https://github.com/user-attachments/assets/b165adef-3c12-4b55-8771-da701ad829a6" />

### Step 4: Test single-quote escaping

Replace the search input with:

```
test'payload
```

Send the request.

Observe that the application escapes the single quote.

Conceptually, the reflected JavaScript becomes:

```
var searchQuery = 'test\'payload';
```

The single quote is now interpreted as part of the string rather than as its closing delimiter.

<img width="943" height="591" alt="Screenshot 2026-11-05 at 8 57 36 PM" src="https://github.com/user-attachments/assets/322c40c5-d16d-4e13-aa42-640703f246cf" />



**Observation:** Directly injecting a single quote does not successfully break out of the string.

### Step 5: Test backslash handling

Now replace the input with:

```
test\payload
```

Send the request and inspect the response.


<img width="947" height="624" alt="Screenshot 2026-11-05 at 9 01 11 PM" src="https://github.com/user-attachments/assets/96c6bc4d-4640-4df9-b0f5-812e8548d2be" />


Observe that the backslash is not escaped by the application.

This is the critical weakness.

The application escapes single quotes but does not correctly handle the character that controls how those quotes are interpreted.

### Step 6: Construct the final payload

Use the following payload:

```
\'-alert(1)//
```

Submit it through the search parameter.

The payload is designed to:

1. Use a backslash to neutralize the application's escaping of the single quote.
2. Terminate the original JavaScript string.
3. Execute `alert(1)`.
4. Use `//` to comment out the remainder of the JavaScript line.

### Step 7: Verify the exploit

1. Send the final request through Burp Repeater.
2. Right-click the request.
3. Select **Copy URL**.
4. Paste the URL into your browser.
5. Load the page.
6. Observe the alert.

<img width="942" height="626" alt="Screenshot 2026-11-05 at 9 07 14 PM" src="https://github.com/user-attachments/assets/5030e191-4909-4842-a921-6534cbfe0a00" />



If the alert executes, the reflected XSS vulnerability has been successfully exploited.


<img width="1150" height="639" alt="Screenshot 2026-11-05 at 9 17 34 PM" src="https://github.com/user-attachments/assets/5f35e7c2-c06e-4830-bfb6-74ec1fabd6d5" />


## 

## 5. Why Does This Work?

The application applies escaping to single quotes without first safely handling backslashes.

In JavaScript, a backslash is an escape character. It affects how the next character is interpreted.

If an application inserts untrusted input into a JavaScript string and fails to escape backslashes consistently, an attacker may manipulate the resulting sequence of characters.

The security failure is therefore not simply that a single quote is allowed. It is that the application uses incomplete and incorrectly ordered escaping.

## 6. Root Cause

The root cause is improper context-specific output encoding.

The application:

- Encodes certain HTML-sensitive characters.
- Escapes single quotes.
- Fails to escape backslashes.
- Inserts untrusted input directly into executable JavaScript.

HTML encoding and JavaScript string encoding solve different problems. Applying one does not automatically protect the other.

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

**1. Avoid embedding untrusted input directly into JavaScript**

Do not construct executable JavaScript source by concatenating user-controlled values.

- **. Use safe serialization**

Use a reliable JSON serializer when transferring server-side values into JavaScript. Ensure that values are safely embedded in the surrounding HTML context as well.

**3. Escape backslashes and quotes correctly**

If JavaScript string encoding is unavoidable, use a context-aware encoder that handles all relevant characters and escaping rules correctly.

**4. Prefer safe DOM APIs**

Use `textContent` for displaying user-controlled text rather than generating HTML or executable JavaScript dynamically.

**5. Implement Content Security Policy**

A restrictive CSP can provide additional defense against script injection, but it does not replace correct output encoding.

##
