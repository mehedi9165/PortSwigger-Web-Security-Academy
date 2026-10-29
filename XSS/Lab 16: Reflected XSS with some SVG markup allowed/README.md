# PortSwigger Lab: Reflected XSS with Some SVG Markup Allowed

## 1. Lab Overview

- **Vulnerability:** Reflected Cross-Site Scripting (XSS)
- **Injection point:** Search parameter
- **Difficulty:** Practitioner
- **Security control:** HTML/SVG tag and event-handler filtering
- **Objective:** Bypass the filter and execute `alert(1)`.

### Learning objectives

- Identify SVG elements allowed by an application filter.
- Use Burp Suite Intruder to test HTML tags and event-handler attributes.
- Understand SVG animation elements and the `onbegin` event.
- Construct a reflected XSS payload using permitted markup.

## 2. Vulnerability Description

The application reflects user-controlled search input into an HTML response. It blocks common XSS tags and attributes but allows certain SVG elements and an event handler that can execute JavaScript.

The vulnerability occurs because the filtering mechanism does not adequately prevent untrusted input from introducing executable markup.

A standard payload such as:

```
<img src=1 onerror=alert(1)>
```

is blocked. Therefore, instead of repeatedly trying common payloads, we use Burp Intruder to systematically identify permitted tags and event handlers.

## 3. Tools Required

- PortSwigger Web Security Academy
- Burp Suite Proxy
- Burp Suite Intruder
- PortSwigger XSS Cheat Sheet

## 4. Step-by-Step Exploitation

### Step 1: Confirm that a common payload is blocked

Enter this payload into the search field:

```
<img src=1 onerror=alert(1)>
```

Submit the search and observe the response.

**Observation:** The request is blocked, indicating that this tag-and-event combination does not pass the application's filter.


<img width="1151" height="570" alt="Screenshot 2026-10-29 at 8 29 06 AM" src="https://github.com/user-attachments/assets/24beb5cd-f24d-49d5-931e-7c7392bbe0cf" />


### Step 2: Send the request to Burp Intruder

1. Open the lab using Burp's browser.
2. Perform a normal search.
3. Find the request in **Proxy → HTTP history**.
4. Right-click it and select **Send to Intruder**.
5. Open Intruder and locate the search parameter.
6. Replace the search value with:

```
<>
```

1. Place the cursor between the angle brackets and click **Add §**.

The payload position should look like:

```
<§§>
```



<img width="1275" height="688" alt="Screenshot 2026-10-29 at 8 54 45 AM" src="https://github.com/user-attachments/assets/530fbcad-b207-457b-9903-d31879a86473" />


<img width="1276" height="758" alt="Screenshot 2026-10-29 at 8 54 31 AM" src="https://github.com/user-attachments/assets/54ab8b18-ecad-4c9b-9f85-3705995f20f9" />



### Step 3: Discover the permitted SVG tags

1. Open the PortSwigger XSS Cheat Sheet.
2. Click **Copy tags to clipboard**.
3. Return to Intruder.
4. In **Payloads → Payload configuration**, click **Paste**.
5. Click **Start attack**.
6. Review the status codes.

According to the lab's solution, these tags receive `200` responses:

- `svg`
- `animatetransform`
- `title`
- `image`

Most other tested tags receive `400` responses.

**Interpretation:** The four tags are promising candidates for further testing. A `200` response alone does not prove that the tag can execute JavaScript.


### Step 4: Discover the permitted event handler

Replace the search value with:

```
<svg><animatetransform%20=1>
```

Place the payload position immediately before the equals sign. It should look like:

```
<svg><animatetransform%20§§=1>
```

Then:

1. Return to the XSS Cheat Sheet.
2. Click **Copy events to clipboard**.
3. In Intruder's **Payloads** panel, click **Clear**.
4. Click **Paste** to load the event-handler list.
5. Click **Start attack**.
6. Review the results.


<img width="1272" height="686" alt="Screenshot 2026-10-29 at 9 06 02 AM" src="https://github.com/user-attachments/assets/6dc411ea-5c03-47ed-bb14-92fc61f81bab" />



<img width="1278" height="710" alt="Screenshot 2026-10-29 at 9 05 47 AM" src="https://github.com/user-attachments/assets/9d7d2335-058e-467e-9c12-4f47109dc77f" />




**Observation:** The lab identifies `onbegin` as the event handler that receives a `200` response, while the other tested event handlers receive `400` responses.

This suggests that the `onbegin` attribute on the permitted SVG animation element is the promising combination to test.

### Step 5: Submit the final payload

Replace `YOUR-LAB-ID` with your actual lab identifier and visit this URL:

```
https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```

The decoded search input is:

```
"><svg><animatetransform onbegin=alert(1)>
```

The payload attempts to close the existing HTML attribute, introduce an SVG element, and add an animation element with an `onbegin` event handler.

If the payload executes successfully, the browser displays an alert and the lab is solved.




<img width="802" height="480" alt="Screenshot 2026-10-29 at 9 18 33 AM" src="https://github.com/user-attachments/assets/1bfc633a-86a4-4f4e-9b2a-3c58eece42fd" />


<img width="1152" height="636" alt="Screenshot 2026-10-29 at 9 12 53 AM" src="https://github.com/user-attachments/assets/d15e1eca-171a-4fad-93ef-c3a64701cfe7" />



## 6. Why the Filter Is Bypassed

The filter blocks many common tags and event handlers but allows a specific combination of SVG markup and an event attribute.

The discovery process separates the problem into two tests:

1. **Tag discovery:** Identify elements that are not blocked.
2. **Event-handler discovery:** Identify attributes that are not blocked.

The final payload combines the permitted `svg` and `animatetransform` elements with `onbegin`.

This demonstrates a weakness of blacklist-based filtering: an application may block familiar XSS patterns while still allowing less common executable markup.

## 7. Security Impact

Reflected XSS can allow attacker-controlled JavaScript to execute in the context of the vulnerable application's origin. Depending on the application, this may allow page manipulation, unauthorized actions performed as the victim, or access to information available to JavaScript.

The actual impact depends on the application's functionality, permissions, and browser protections.

## 8. Remediation

- Apply context-aware output encoding to user-controlled search input.
- Avoid inserting untrusted values directly into HTML markup.
- Use safe DOM APIs such as `textContent` when displaying plain text.
- If HTML input is required, use a maintained sanitizer with a restrictive allowlist that handles SVG safely.
- Use Content Security Policy as an additional defense.
- Do not rely exclusively on blocklists of HTML tags or event-handler names.
