# DOM XSS in `document.write()` Sink Using `location.search` Inside a Select Element

## 1. Vulnerability

**Vulnerability:** DOM-Based Cross-Site Scripting (DOM XSS)

**Difficulty:** Apprentice

**Source:** `location.search`

**Sink:** `document.write()`

**Injection Context:** HTML inside a `<select>` element

**Affected Functionality:** Product stock checker

**Payload:**

```html
"></select><img src=1 onerror=alert(1)>
```

### Description

The application contains a DOM-based XSS vulnerability in its stock checker functionality.

Client-side JavaScript reads the `storeId` parameter from `location.search` and uses `document.write()` to insert the value into the page. The inserted content is intended to represent an option within a `<select>` element.

Because the application inserts attacker-controlled data into HTML without appropriate validation or encoding, an attacker can terminate the existing HTML attribute and close the `<select>` element. The injected `<img>` element then triggers its `onerror` event when the image fails to load, executing `alert(1)`.

This vulnerability demonstrates the risks of writing untrusted URL parameters directly into HTML.

---

## 2. Objective

- Identify `location.search` as the attacker-controlled source.
- Trace the `storeId` parameter through client-side JavaScript.
- Identify `document.write()` as the vulnerable sink.
- Understand the HTML context surrounding the injected value.
- Escape the existing attribute and `<select>` element.
- Inject an HTML element that executes JavaScript.
- Understand appropriate remediation techniques.

---

## 3. Step-by-Step Exploitation

### Step 1 — Open a product page

Open the lab and navigate to a product page.

Locate the **Check stock** or stock checker functionality.

The page contains a drop-down list used to select a store.

### Step 2 — Identify the `storeId` parameter

Inspect the page's URL.

The product URL may look similar to:

```text
/product?productId=1
```

The lab's JavaScript reads a query parameter named:

```text
storeId
```

The relevant browser API is:

```javascript
location.search
```

This property contains the query-string portion of the current URL.

For example:

```text
?productId=1&storeId=London
```

The parameter `storeId` can be controlled by changing the URL.

### Step 3 — Submit a random test value

Add a `storeId` parameter containing a unique alphanumeric string:

```text
?productId=1&storeId=test
```

Load the modified URL.



### Step 4 — Observe the drop-down list

Look at the stock checker drop-down list.

If the value is reflected successfully, you should see `test` as one of the options.

This confirms that the parameter influences the page's client-side output.


<img width="1325" height="746" alt="Screenshot 2026-10-16 at 8 58 06 AM" src="https://github.com/user-attachments/assets/956eda84-317d-40c4-ad52-c373bd999b13" />


### Step 5 — Inspect the HTML

Right-click the drop-down list and select **Inspect**.

Examine the generated `<select>` and `<option>` elements.

You may observe markup conceptually similar to:

```html
<select>
    <option>London</option>
    <option>Paris</option>
    <option>test123</option>
</select>
```


<img width="880" height="536" alt="Screenshot 2026-10-16 at 12 32 50 AM" src="https://github.com/user-attachments/assets/0eb0e846-b6d1-459a-88e9-ac1de01b3ba5" />


The important finding is that attacker-controlled input has been inserted into an HTML structure.



<img width="883" height="520" alt="Screenshot 2026-10-16 at 12 21 21 AM" src="https://github.com/user-attachments/assets/ae100802-f79f-4305-b7c2-48263eac296d" />



<img width="1324" height="743" alt="Screenshot 2026-10-16 at 12 21 53 AM" src="https://github.com/user-attachments/assets/01f2c03e-d16a-4cb0-a338-9c58c0e1bbde" />

### Step 6 — Understand the vulnerable JavaScript

The application reads a value from the query string and passes it to `document.write()`.


The vulnerable data flow is:

```text
location.search
       ↓
storeId
       ↓
document.write()
       ↓
HTML parsing
       ↓
Stock checker DOM
```

`document.write()` writes its supplied string as HTML rather than automatically treating it as plain text.

### Step 7 — Construct the XSS payload

Use the following payload:

```html
"></select><img src=1 onerror=alert(1)>
```


### Step 8 — Insert the payload into the URL

Use the product URL with the `storeId` parameter:

```text
/product?productId=1&storeId="></select><img src=1 onerror=alert(1)>
```


Load the modified URL in the lab browser.

<img width="883" height="538" alt="Screenshot 2026-10-16 at 12 30 51 AM" src="https://github.com/user-attachments/assets/0f78ac4e-c228-4b96-a220-0629d22221b9" />



### Step 9 — Verify JavaScript execution

If the payload executes successfully, the browser displays a JavaScript alert containing:

```text
1
```

This confirms that the injected HTML was interpreted by the browser and that the `onerror` handler executed.

---



<img width="1322" height="745" alt="Screenshot 2026-10-16 at 12 31 04 AM" src="https://github.com/user-attachments/assets/8b32c3e9-1b6a-4b3e-bff5-9b19a2ac4de0" />


## 4. Payload Breakdown

The complete payload is:

```html
"></select><img src=1 onerror=alert(1)>
```

### `"`

The double quote terminates the existing quoted attribute.

### `>`

The greater-than sign closes the current HTML tag.

### `</select>`

This closes the surrounding drop-down element, allowing subsequent markup to be interpreted outside that element.

### `<img src=1>`

Creates an image element. The browser attempts to load the resource specified by `src`.

### `onerror=alert(1)`

When image loading fails, the `onerror` handler invokes `alert(1)`.


---

## 5. Why Does This Work?

The vulnerability exists because the application treats untrusted query-string data as HTML.

The source is:

```javascript
location.search
```

The dangerous sink is:

```javascript
document.write()
```

When attacker-controlled data reaches this sink, the browser parses it as markup.

The application does not correctly prevent the input from changing the surrounding HTML structure.

**Key lesson:** Untrusted input should not be inserted directly into HTML using `document.write()`.

---

## 6. Impact

Depending on the application's functionality and the victim's privileges, DOM-based XSS may allow an attacker to:

- Execute JavaScript in the vulnerable application's origin.
- Modify page content.
- Perform actions available to the victim.
- Access information exposed to JavaScript.
- Present phishing content within a trusted interface.
- Target privileged users.

The actual impact depends on the application's architecture, browser protections, authentication model, and other security controls.

---

## 7. Defensive Measures

### 7.1 Avoid `document.write()` for untrusted data

Avoid inserting untrusted URL parameters into HTML strings.

Where possible, create DOM elements using safe APIs and assign user-controlled values as text.

For example:

```javascript
const option = document.createElement('option');
option.textContent = storeId;
selectElement.appendChild(option);
```

This treats `storeId` as text instead of parsing it as HTML.

The application should also validate that `storeId` corresponds to a legitimate store identifier.

### 7.2 Validate the `storeId` parameter

If `storeId` should be a numeric identifier or one of a known set of store IDs, enforce that expected format.

Do not rely on input validation alone to prevent XSS; safe output handling is also necessary.

### 7.3 Use context-appropriate output handling

When data must be inserted into HTML, encode it for the correct context or use APIs that avoid HTML parsing.

HTML text, HTML attributes, JavaScript strings, and URLs require different handling.

### 7.4 Implement Content Security Policy

A strong Content Security Policy (CSP) can provide additional protection against XSS.

CSP is a defense-in-depth measure and should not replace safe DOM manipulation.

---

