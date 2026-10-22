# DOM XSS in AngularJS Expression with Angle Brackets and Double Quotes HTML-Encoded

## 1. Vulnerability

**Vulnerability:** DOM-Based Cross-Site Scripting (DOM XSS)

**Difficulty:** Apprentice

**Injection Point:** Search functionality

**Technology:** AngularJS

**Source:** User-controlled search input

**Execution Context:** AngularJS expression inside an `ng-app` region

**Payload:**

```
{{$on.constructor('alert(1)')()}}
```

### Description

The application contains a DOM-based XSS vulnerability in its search functionality.

The search input is reflected into a section of the page that AngularJS processes. This section is associated with the `ng-app` directive, which tells AngularJS to compile and process AngularJS directives and expressions within that application region.

AngularJS evaluates expressions enclosed in double curly braces:

```
{{ expression }}
```

Although the application HTML-encodes angle brackets and double quotes, these protections do not prevent AngularJS from evaluating an expression that has been inserted into an AngularJS-enabled region.

The payload uses the `constructor` property to obtain a function constructor and execute JavaScript that calls `alert(1)`.

This demonstrates that **HTML encoding alone does not protect an application that evaluates untrusted input as a client-side template expression**.

---

## 2. Objective

The objective of this lab is to:

- Identify where the search input is reflected.
- Determine whether the reflected input appears inside an AngularJS application region.
- Understand the purpose of the `ng-app` directive.
- Identify AngularJS expression interpolation using `{{ }}`.
- Execute an AngularJS expression that calls `alert(1)`.
- Understand why HTML encoding does not prevent this vulnerability.
- Identify appropriate defensive measures.

---

## 3. Step-by-Step Exploitation

### Step 1 — Open the lab

Open the PortSwigger lab and navigate to the home page.

Locate the search input field.

### Step 2 — Enter a random alphanumeric string

Enter a unique test value into the search box:

```
test123abc
```

Click **Search**.

The random string helps you identify where the application places your input.

### Step 3 — Inspect the page source

Right-click the page and select **View Page Source**.

Search for:

```
test123abc
```

Inspect the surrounding HTML.

You should find that your input is reflected in a region associated with the AngularJS `ng-app` directive.

The exact HTML structure should be recorded from your own lab response.

<img width="1320" height="746" alt="Screenshot 2026-10-22 at 7 57 28 PM" src="https://github.com/user-attachments/assets/2599f051-96bc-4245-b14a-0541addee073" />



### Step 4 — Enter the AngularJS payload

Replace the random search string with:

```
{{$on.constructor('alert(1)')()}}
```

Click **Search**.

### Step 5 — Understand what happens

AngularJS processes the expression within the `ng-app` region.

The payload uses JavaScript's `constructor` property to obtain a function constructor, then invokes the resulting function to execute:

```
alert(1)
```

If the vulnerable application evaluates the expression as expected, a JavaScript alert containing `1` appears.



<img width="1320" height="746" alt="Screenshot 2026-10-22 at 8 07 27 PM" src="https://github.com/user-attachments/assets/d92d0ce9-7cd8-4a01-8651-3d4d40316e68" />


## 4. Understanding the Payload

The payload is:

```
{{$on.constructor('alert(1)')()}}
```

It has several components.

### `{{ ... }}` — AngularJS expression interpolation

AngularJS evaluates expressions placed inside double curly braces.

For example:

```
<p>{{ 2 + 3 }}</p>
```

renders:

```
5
```

In this lab, the attacker uses the same expression mechanism to trigger JavaScript execution.

### `$on`

`$on` is commonly associated with AngularJS scope event-listener functionality. In this payload, the relevant purpose is to access its inherited `constructor` property.

### `.constructor`

In JavaScript, functions and other objects inherit properties through their prototype chains. The `constructor` property can provide access to the function constructor in the relevant context.

### `('alert(1)')`

This supplies the JavaScript source string that the function constructor will use.

### `()`

The final pair of parentheses invokes the constructed function.

---

## 5. Why Does This Work?

The root cause is that untrusted input is placed into an area where AngularJS evaluates expressions.

The application may encode HTML characters such as:

```
<  →  &lt;
>  →  &gt;
"  →  &quot;
```

However, AngularJS expression syntax does not require an injected HTML element.

The payload uses:

```
{{ ... }}
```

which AngularJS recognizes as expression interpolation.

Therefore, encoding angle brackets and double quotes does not prevent the framework from evaluating the expression.

**Key lesson:** HTML output encoding does not neutralize untrusted AngularJS template expressions.

---

## 

---

## 6. Impact

Depending on the application and its configuration, AngularJS expression injection may allow an attacker to:

- Execute JavaScript in the application's origin.
- Modify webpage content.
- Perform actions available to the victim.
- Access information exposed to JavaScript.
- Present phishing content within the trusted application.
- Target privileged users.

The actual impact depends on the AngularJS version, the expression evaluation context, the application's functionality, and other security controls.

---

## 7. Defensive Measures

### 7.1 Never compile untrusted input as an AngularJS template

The most important defense is to prevent user-controlled data from being interpreted as AngularJS expressions or templates.

Treat search terms as data, not executable template content.

### 7.2 Avoid dynamically compiling user-controlled HTML

Do not pass untrusted content into AngularJS compilation mechanisms such as `$compile`.

If rich HTML content is genuinely required, apply a carefully configured HTML sanitization policy and avoid evaluating AngularJS expressions within that content.

### 7.3 Use safe text rendering

Render user-controlled search terms as text instead of template source.

For example, AngularJS applications can use a normal binding:

```
<p>{{ searchTerm }}</p>
```

Here, `searchTerm` should be a data value supplied to the template, not a string that the application subsequently compiles as a new AngularJS template.

The distinction is important: ordinary interpolation of a data value is different from compiling attacker-controlled template markup.

### 7.4 Keep AngularJS Updated

AngularJS is a legacy framework, and official support for AngularJS 1.x has ended. Where possible, migrate to a currently supported framework and follow its security guidance.

### 7.5 Implement Defense in Depth

Use appropriate output encoding, input validation, secure framework practices, and a strong Content Security Policy (CSP).

These measures complement the primary defense of preventing untrusted input from being interpreted as executable expressions.
