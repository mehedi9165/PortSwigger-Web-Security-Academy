# PortSwigger Lab: Reflected XSS into HTML Context with Most Tags and Attributes Blocked

## 1. Lab Overview

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Difficulty:** Practitioner

**Injection point:** Search functionality

**Security control:** Web Application Firewall (WAF)

**Objective:** Bypass the WAF and execute JavaScript's `print()` function without requiring user interaction.

### Learning objectives

- Understand how a WAF blocks common XSS payloads.
- Use Burp Suite Intruder to identify permitted HTML tags and event handlers.
- Interpret HTTP response codes during payload discovery.
- Construct an XSS payload using an allowed tag and event handler.
- Deliver an exploit through the PortSwigger exploit server.

## 2. Vulnerability Description

The application's search functionality reflects user-controlled input into an HTML response. Although a WAF blocks many common HTML tags and JavaScript event handlers, it does not block every possible combination.

A standard payload such as:

```
<img src=1 onerror=print()>
```

is blocked by the WAF.

The vulnerability exists because the application allows a combination of HTML markup and an event handler that can execute JavaScript when a particular browser event occurs.

**Security lesson:** A WAF is a defense-in-depth control, not a replacement for context-aware output encoding and safe handling of untrusted input.

## 3. Tools Required

- PortSwigger Web Security Academy lab
- Burp Suite
- Burp Proxy
- Burp Intruder
- PortSwigger XSS Cheat Sheet
- PortSwigger Exploit Server

## 4. Step-by-Step Exploitation

### Step 1: Test a standard XSS payload

Enter the following payload into the search field:

```
<img src=1 onerror=print()>
```

Submit the search.

**Observation:** The request is blocked, demonstrating that the WAF rejects this common combination of an HTML tag and an event handler.

A blocked payload does not necessarily mean that the underlying application is secure. Other HTML elements and event handlers may remain available.


<img width="1277" height="713" alt="Screenshot 2026-10-26 at 9 47 02 PM" src="https://github.com/user-attachments/assets/1a589a7b-8369-4a6e-8e57-29f711242bea" />



### Step 2: Send the request to Burp Intruder

1. Open the lab in Burp's browser.
2. Submit a normal search query.
3. Locate the search request in **Proxy → HTTP history**.
4. Right-click the request and select **Send to Intruder**.
5. Open the Intruder tab and locate the search parameter.

Replace the search value with:

```
<>
```

Place the cursor between the angle brackets and click **Add §**.

The resulting payload position should resemble:

```
<§§>
```

The markers identify where Intruder will insert each test payload.

<img width="1278" height="689" alt="Screenshot 2026-10-27 at 10 17 23 AM" src="https://github.com/user-attachments/assets/fdc59586-c2e9-4ab6-8422-595a9f08acd0" />


<img width="1278" height="758" alt="Screenshot 2026-10-27 at 10 28 18 AM" src="https://github.com/user-attachments/assets/f231ead8-2859-4e48-8d13-4bf61c65d086" />



### Step 3: Discover allowed HTML tags

1. Open the PortSwigger XSS Cheat Sheet:
    
    https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
    
2. Select **Copy tags to clipboard**.
3. Return to Burp Intruder.
4. Under **Payloads → Payload configuration**, click **Paste**.
5. Start the attack.
6. Review the HTTP status codes in the results.

**Observed result in this lab:** Most tested tags return `400`, while the `body` tag returns `200`.

This indicates that the WAF treats the tested tags differently. A `200` response is a promising candidate, but it does not by itself prove that a payload executes.

### Step 4: Discover allowed event handlers

Change the search value to:

```
<body%20=1>
```

Place the payload marker immediately before the equals sign so the value resembles:

```
<body%20§§=1>
```

Next:

1. Open the XSS Cheat Sheet.
2. Click **Copy events to clipboard**.
3. Return to Burp Intruder.
4. Clear the previous payload list.
5. Paste the event-handler list.
6. Start the attack.
7. Review the HTTP responses.



<img width="1279" height="758" alt="Screenshot 2026-10-27 at 10 30 14 AM" src="https://github.com/user-attachments/assets/5b9c516e-ba8b-42da-9154-7043e965b3ad" />




<img width="1278" height="759" alt="Screenshot 2026-10-27 at 10 40 29 AM" src="https://github.com/user-attachments/assets/62108395-94f9-4fe2-a465-6a2d733f131e" />




**Observed result in this lab:** Most event handlers return `400`, while `onresize` returns `200`.

This identifies `onresize` as a promising event handler for the allowed `body` element.

### Step 5: Construct the final payload

The payload uses the following HTML structure:

```
"><body onresize=print()>
```

The initial double quote closes the quoted HTML attribute into which the search input is reflected. The `>` then attempts to terminate that element, allowing the new `body` element to be interpreted as markup.

The `onresize` event handler calls `print()` when a resize event occurs.


<img width="1156" height="640" alt="Screenshot 2026-10-27 at 12 09 28 PM" src="https://github.com/user-attachments/assets/a8e70828-283b-4753-900d-30f451feba87" />



### Step 6: Deliver the exploit to the victim

On the PortSwigger Exploit Server, enter the following HTML, replacing `YOUR-LAB-ID` with the actual lab identifier:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

Then:

1. Click **Store**.
2. Click **Deliver exploit to victim**.
3. Check the lab status to confirm whether it has been solved.

The iframe's `onload` handler changes its width to `100px`. That resize can trigger the injected `body` element's `onresize`handler in the victim's browser, calling `print()` without requiring the victim to click anything.

**Important:** The exploit must run in the victim's browser through the exploit server. Manually calling `print()` in your own browser is not sufficient to solve the lab.


<img width="1152" height="639" alt="Screenshot 2026-10-27 at 8 55 54 PM" src="https://github.com/user-attachments/assets/c043026d-de0a-4e53-ba4e-a8ae51f054a3" />



## 5. Why the WAF Bypass Works

The discovery process tests two separate components:

1. **HTML tag filtering:** Identifies which tags the WAF allows through.
2. **Event-handler filtering:** Identifies which event attributes the WAF allows through.

The `body` tag and `onresize` event handler form a combination that passes the lab's filtering rules.

The exploit then uses HTML parsing and a browser-generated resize event to execute the JavaScript function.

A WAF that relies primarily on blocklists may fail to recognize less common combinations of otherwise valid HTML features.

## 6. Root Cause and Defensive Recommendations

### Root cause

The application reflects untrusted search input into an HTML response without reliably preventing the input from changing the HTML structure or introducing executable event handlers.

### Recommended defenses

- **Context-aware output encoding:** Encode untrusted values according to the HTML context in which they are inserted.
- **Avoid unsafe HTML construction:** Prefer safe DOM APIs such as `textContent` when rendering plain text.
- **Use a vetted HTML sanitizer:** If user-supplied HTML is genuinely required, apply a maintained sanitizer with a restrictive allowlist.
- **Apply a Content Security Policy (CSP):** Use CSP as an additional defense, not as the primary fix.
- **Retain the WAF as defense in depth:** Update and test its rules, but do not rely on it to eliminate XSS vulnerabilities.

## 7. Impact

If exploited against a vulnerable application, reflected XSS can execute attacker-controlled JavaScript in the context of the affected site's origin. Depending on the application's design and available browser protections, this may enable actions such as:

- Manipulating page content.
- Performing unauthorized actions as the victim.
- Accessing data exposed to scripts within the page.
- Redirecting or deceiving users.

The actual impact depends on the application's authentication model, permissions, browser protections, and the data accessible to JavaScript.
