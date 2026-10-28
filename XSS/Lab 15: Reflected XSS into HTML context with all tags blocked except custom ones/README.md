# PortSwigger Lab: Reflected XSS into HTML Context with All Tags Blocked Except Custom Ones

## 1. Lab Overview

**Vulnerability:** Reflected Cross-Site Scripting (XSS)

**Injection point:** Search functionality

**Security control:** HTML tag filtering that blocks standard tags but permits custom elements

**Objective:** Inject a custom HTML element that automatically executes `alert(document.cookie)` without requiring user interaction.

### Learning objectives

- Understand custom HTML elements and HTML tag filtering.
- Understand the `onfocus` event handler.
- Learn how `tabindex` makes an element programmatically focusable.
- Understand how a URL fragment can trigger focus on an element.
- Distinguish between JavaScript executed by the exploit server and JavaScript executed in the vulnerable application.

## 2. Vulnerability Description

The application reflects the search parameter into an HTML response. Its filtering mechanism blocks standard HTML tags but permits custom element names.

An attacker can exploit this behavior by injecting a custom element with an event handler. When the browser focuses that element, the event handler executes JavaScript.

The key issue is not simply that custom elements are allowed. The vulnerability occurs because untrusted input can introduce an executable event handler into the page's HTML.

## 3. Exploit Payload

The exploit server contains the following HTML:

```
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

Replace `YOUR-LAB-ID` with the identifier of your own lab.

The URL decodes to the following search input:

```
<xss id=x onfocus=alert(document.cookie) tabindex=1>
```

The final URL also includes the fragment:

```
#x
```

## 4. Step-by-Step Exploitation

### Step 1: Open the lab

Open your PortSwigger Web Security Academy lab. If the lab fails to start and reports a timeout, retry it later; this is an availability problem and does not necessarily indicate an issue with your payload.

### Step 2: Understand the search parameter

The vulnerable parameter is `search`. The payload injects a custom tag named `xss` rather than a standard tag such as I`<img>`or `<script>`.

The custom element includes an `id`, an `onfocus` handler, and a `tabindex` attribute.

### Step 3: Check the exploit

In the search parameter inject following

```
<xss id=x onfocus=alert(document.cookie) tabindex=1>
```

<img width="1154" height="481" alt="Screenshot 2026-10-28 at 12 19 52 PM" src="https://github.com/user-attachments/assets/45fba598-d41f-469b-a0af-a92145bf7243" />


Now add following with the URL for focus and check the output:

```
#x
```

<img width="1152" height="640" alt="Screenshot 2026-10-28 at 12 25 32 PM" src="https://github.com/user-attachments/assets/f6521b5e-40b0-4ddc-af77-3c24a4e7049c" />


<img width="1153" height="635" alt="Screenshot 2026-10-28 at 12 26 20 PM" src="https://github.com/user-attachments/assets/5fcbd7ab-56c0-4ee4-bf68-d8f3e5bdc6e3" />



### Step 4: Open the exploit server

Open the lab's linked exploit server and paste the exploit code into its body field.

```
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

Replace the placeholder with your actual lab hostname, then click **Store**.


<img width="1148" height="640" alt="Screenshot 2026-10-28 at 12 27 38 PM" src="https://github.com/user-attachments/assets/3e520e7f-27b0-492f-b0b9-0aa538004aa6" />


### Step 5: Deliver the exploit

Click **Deliver exploit to victim**.

When the victim visits the exploit page, the script navigates the victim's browser to the vulnerable lab URL. The browser processes the injected custom element, and the URL fragment targets the element with `id="x"`.

### Step 6: Verify the result

Check the lab status. A successful exploit should trigger the JavaScript alert automatically in the victim's browser, without requiring a click.

## 5. Why Is `<script>` Used on the Exploit Server?

This is an important distinction.

The exploit-server page contains:

```
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=...#x';
</script>
```

The `<script>` element executes on the **exploit server's page**, not inside the vulnerable application's search parameter.

Its purpose is to navigate the browser to the malicious URL. The vulnerable application receives the search input, and its HTML response is where the custom element and event handler are interpreted.

Therefore, the two contexts are different:

1. **Exploit server:** The script performs browser navigation.
2. **Vulnerable application:** The reflected custom element triggers the XSS behavior.

The lab's filter on HTML tags in the search input does not automatically mean that the exploit server's separate HTML page is subject to the same filter.

## 6. Why the Attack Requires No User Interaction

The exploit combines three mechanisms:

1. `tabindex=1` makes the custom element focusable.
2. `#x` targets the element by its ID.
3. `onfocus` runs the JavaScript when focus occurs.

This is why the payload does not rely on the victim clicking the custom element.

Browser behavior can vary with focus handling and document structure, so the lab's completion status is the definitive confirmation that the particular payload worked in the target environment.

## 7. Security Impact

Reflected XSS can execute attacker-controlled JavaScript in the context of the vulnerable application's origin.

Depending on the application and browser protections, this may allow an attacker to manipulate page content, perform actions as the victim, or access data exposed to scripts.

`document.cookie` does not expose cookies marked `HttpOnly`, so the payload cannot read those cookies through JavaScript. The lab's alert is a demonstration of script execution, not proof that every session cookie is accessible.

## 8. Remediation

- Apply context-aware output encoding to untrusted search input.
- Avoid inserting untrusted data directly into HTML markup.
- Use safe DOM APIs such as `textContent` when displaying plain text.
- If user-supplied HTML is required, use a maintained HTML sanitizer with a restrictive allowlist.
- Use a suitable Content Security Policy as an additional layer of defense.
- Do not rely solely on blocking standard HTML tag names; custom elements and event-handler attributes must also be considered.
