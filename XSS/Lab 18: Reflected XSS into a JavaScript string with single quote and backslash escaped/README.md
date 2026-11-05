# PortSwigger Lab: Reflected XSS into a JavaScript String with Single Quote and Backslash Escaped

## 1. Lab Overview

- **Vulnerability:** Reflected Cross-Site Scripting (XSS)
- **Injection point:** Search functionality
- **Injection context:** JavaScript string inside a script block
- **Difficulty:** Practitioner
- **Security mechanism:** Single quotes and backslashes are escaped
- **Objective:** Break out of the JavaScript context and execute `alert(1)`.

### Learning objectives

- Understand how user input can be reflected inside JavaScript strings.
- Understand single-quote and backslash escaping.
- Identify the limitations of JavaScript string escaping.
- Learn how to terminate an HTML script element.
- Understand the difference between breaking out of a JavaScript string and breaking out of an HTML script block.

## . Vulnerability Description

The application contains a search query tracking functionality that reflects the user's search input inside a JavaScript string.

The vulnerable implementation is conceptually similar to:

```
<script>
    var searchQuery = 'USER_INPUT';
</script>
```

The application attempts to protect this functionality by escaping:

- Single quotes (`'`)
- Backslashes ()

## 3. Tools Required

- PortSwigger Web Security Academy
- Burp Suite Proxy
- Burp Suite Repeater
- Browser Developer Tools

## 4. Step-by-Step Exploitation

### Step 1: Submit a normal search query

Open the lab and enter a random alphanumeric string into the search box.

For example:

```
test123
```

Submit the search request.



<img width="804" height="576" alt="Screenshot 2026-11-05 at 2 06 20 PM" src="https://github.com/user-attachments/assets/a93afef7-b2cb-41f8-ab82-80b14447318a" />

### Step 2: Intercept the request using Burp Suite

1. Open Burp Suite.
2. Navigate to **Proxy → HTTP history**.
3. Find the search request.
4. Right-click the request.
5. Select **Send to Repeater**.

### Step 3: Inspect the reflected input

Open the response in Burp Repeater.

Search for the random string you submitted.

You should find that the input is reflected inside a JavaScript string in a script block.

Conceptual example:

```
<script>
    var searchQuery = 'test123';
</script>
```

This confirms that the application places user-controlled data inside JavaScript code.

### Step 4: Test single-quote escaping

Replace the search input with:

```
test'payload
```

Send the request.

Observe the response.

The application escapes the single quote, producing something similar to:

```
<script>
       var searchTerms = 'test\'payload';
       document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```




<img width="948" height="692" alt="Screenshot 2026-11-05 at 2 09 44 PM" src="https://github.com/user-attachments/assets/48c0024d-b6b0-4cc1-a4fc-369066a3860c" />

**Observation:** The single quote cannot directly terminate the JavaScript string.

This confirms that the application has implemented some escaping, but it does not establish that the entire HTML and JavaScript context is safe.

### Step 5: Break out of the script block

Replace the search input with:

```
</script><script>alert(1)</script>
```

Send the request in Burp Repeater.

The first `</script>` attempts to terminate the existing script element.

The second `<script>` introduces a new script element containing the JavaScript alert.

### Step 6: Verify the reflected XSS

After sending the request:

1. Right-click the request in Burp Repeater.
2. Select **Copy URL**.
3. Paste the URL into your browser.
4. Load the page.
5. Observe whether the alert appears.

If the alert executes, the XSS vulnerability has been successfully demonstrated.


<img width="1274" height="687" alt="Screenshot 2026-11-05 at 2 04 37 PM" src="https://github.com/user-attachments/assets/19d9d5ec-25df-467a-9f6e-99ae2184050b" />



<img width="1155" height="637" alt="Screenshot 2026-11-05 at 2 24 47 PM" src="https://github.com/user-attachments/assets/fa77520c-03a3-470b-bcb7-ab3eba6d3b1d" />

## 5. Detailed Payload Breakdown

### Final payload

```
</script><script>alert(1)</script>
```

| Payload component | Function |
| --- | --- |
| `</script>` | Attempts to terminate the existing HTML script element. |
| `<script>` | Starts a new script element. |
| `alert(1)` | JavaScript function used to demonstrate execution. |
| `</script>` | Closes the newly injected script element. |

### A. Why `</script>` is important

The browser's HTML parser recognizes the closing script tag.

Even if the application escapes single quotes and backslashes inside a JavaScript string, that does not necessarily prevent the HTML parser from recognizing a script-closing sequence.

This is the critical weakness in the lab.

### B. Why the second `<script>` is necessary

After terminating the original script element, the attacker introduces another script element.

```
<script>alert(1)</script>
```

The browser interprets this as a new script block and executes the JavaScript.

## 6. Understanding the Difference Between the Two Injection Techniques

This lab demonstrates two different ways of attacking JavaScript embedded in HTML.

| Technique | What the attacker attempts |
| --- | --- |
| JavaScript string breakout | Escape the string using a quote and inject JavaScript. |
| HTML script-block breakout | Terminate the entire script element and introduce a new one. |

The first technique is blocked by the application's escaping mechanism.

The second technique succeeds because the application does not adequately protect the HTML script-element context.

**Important:** JavaScript string escaping and HTML script-context protection are separate security requirements.

## 

## 7. Security Impact

Successful reflected XSS can allow attacker-controlled JavaScript to execute in the context of the vulnerable application's origin.

Depending on the application, possible consequences include:

- Manipulation of webpage content.
- Unauthorized actions performed as the victim.
- Access to information available to JavaScript.
- Phishing or deceptive content within the trusted website.

The actual impact depends on the application's functionality and security controls.

## 8. Remediation

### Recommended defenses

**1. Avoid embedding untrusted input directly inside JavaScript**

Prefer passing data through a safe data mechanism rather than concatenating it into executable script source.

- **. Use JSON serialization correctly**

If server-side data must be transferred to JavaScript, use a serializer designed for safe embedding in the relevant HTML context. Ensure that HTML-sensitive characters, particularly `<`, are handled safely.

**3. Apply context-aware output encoding**

JavaScript-string encoding alone does not protect the surrounding HTML script element.

**4. Use safe DOM APIs**

When displaying search terms, use `textContent` instead of constructing HTML using untrusted strings.

**5. Implement Content Security Policy**

A restrictive CSP can provide additional protection against script injection, but it should not replace proper output handling.
