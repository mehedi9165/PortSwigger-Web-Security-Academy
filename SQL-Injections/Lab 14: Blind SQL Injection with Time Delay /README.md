# Lab Objective

Verify that the `TrackingId` cookie is vulnerable to SQL injection by causing the database to **pause execution for 10 seconds**.

---

# Step 1: Intercept the Request

1. Open **Burp Suite**.
2. Turn **Intercept ON**.
3. Visit the lab's home page.
4. Capture a request similar to:

```
GET / HTTP/2
Host: lab-id.web-security-academy.net
Cookie: TrackingId=x; session=...
```

Send the request to **Repeater** (`Ctrl + R`).

---

# Step 2: Modify the `TrackingId` Cookie

Replace the cookie value with:

```
TrackingId=x'||pg_sleep(10)--
```

---

<img width="1278" height="725" alt="Screenshot 2026-08-09 at 9 01 45 AM" src="https://github.com/user-attachments/assets/b5a9eb31-dde3-41ef-bd32-9caf6ce1c79f" />


# Breaking Down the Payload

```sql
x'||pg_sleep(10)--
```

### **1. `x`**

This is the original cookie value.

```
TrackingId=x
```

---

### **2. `'`**

Closes the SQL string started by the application.

Example original query:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid='x'
```

After injection:

```sql
WHERE trackingid='x'
```

You have escaped from the string and can inject SQL.

---

### **3. `||`**

In **PostgreSQL**, `||` is the **string concatenation operator**.

Example:

```sql
'Hello' || 'World'
```

returns:

```
HelloWorld
```

Here it is used to append the result of the function call to the existing string expression.

---

### **4. `pg_sleep(10)`**

`pg_sleep()` is a built-in **PostgreSQL** function.

```sql
pg_sleep(10)
```

tells the database:

> "Pause execution for 10 seconds before continuing."
> 

It returns no meaningful value for the user; its purpose here is simply to introduce a measurable delay.

---

### **5. `-`**

Starts a SQL comment.

Everything after `--` is ignored by the database, preventing the remainder of the original SQL statement from causing syntax errors.

---

# What Happens Internally?

Suppose the application executes:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid='x'
```

After your injection, the SQL becomes conceptually similar to:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid='x' || pg_sleep(10)--'
```

The database must execute `pg_sleep(10)` before completing the query.

---

# Expected Result

When you click **Send** in Repeater:

- The request does **not** fail.
- The page content looks normal.
- **However, the response takes approximately 10 seconds longer than usual.**

This confirms that:

- Your input is being interpreted as SQL.
- The injected function executed successfully.
- The backend database is likely **PostgreSQL**, because `pg_sleep()` is a PostgreSQL-specific function.

---

# Why Is This Called Blind SQL Injection?

You do **not** receive:

- SQL error messages.
- Database output.
- Extra data in the response.

The only evidence of successful injection is the **response time**.

---

# Visual Workflow

```
Intercept Request
        │
        ▼
Modify TrackingId
        │
        ▼
TrackingId=x'||pg_sleep(10)--
        │
        ▼
Database executes pg_sleep(10)
        │
        ▼
Execution pauses for 10 seconds
        │
        ▼
Application returns the normal page
        │
        ▼
Measured delay confirms SQL injection
```

# Key Takeaway

The payload:

```
TrackingId=x'||pg_sleep(10)--
```

uses PostgreSQL's `pg_sleep(10)` function to intentionally delay query execution. If the application's response is delayed by roughly **10 seconds**, it indicates that the injected SQL was executed successfully. This is the basis of **time-based blind SQL injection**, where response timing—not page content or error messages—is used to infer whether injected SQL has been executed.
