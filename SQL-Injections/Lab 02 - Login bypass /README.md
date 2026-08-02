# Step 1: Check Vulnerability

In Username:

```
'
```

Output:

- **Internal Server Error**

---

<img width="1278" height="729" alt="Screenshot 2026-08-02 at 11 50 45 PM" src="https://github.com/user-attachments/assets/b281496a-88fd-4e52-a44c-c8cd4c639242" />
<img width="1278" height="724" alt="Screenshot 2026-08-02 at 11 53 02 PM" src="https://github.com/user-attachments/assets/231d7dd2-af04-4be4-97db-2cebdc88115c" />



# Step 2: Edit the username

In username:

```
administrator'--
```

Enter any password

You will get logged in bypassing password
<img width="955" height="452" alt="Screenshot 2026-08-03 at 12 03 26 AM" src="https://github.com/user-attachments/assets/1a7def32-065b-46f4-9b14-ba6ba9334eda" />
<img width="1275" height="656" alt="Screenshot 2026-08-03 at 12 04 03 AM" src="https://github.com/user-attachments/assets/82119b7c-9f0a-4286-9c3f-a19b3b8ceb09" />

### Mitigation

To prevent **SQL Injection Authentication Bypass** vulnerabilities:

* Use **parameterized queries (prepared statements)** instead of concatenating user input into SQL queries.
* Store passwords using **strong salted hashing algorithms** (e.g., Argon2, bcrypt, or PBKDF2) and verify them securely.
* Validate user input using **allow-list validation** where appropriate.
* Return **generic authentication error messages** without revealing database details.
* Apply the **principle of least privilege** to database accounts.
* Enable **Multi-Factor Authentication (MFA)** for sensitive accounts as an additional security layer.

### 💡 Lessons Learned

* User-controlled input should never be trusted.
* SQL Injection can bypass authentication if user input is directly embedded into SQL queries.
* **Prepared statements** are the most effective defense against SQL Injection.
* Passwords should never be stored in plaintext; always use secure password hashing.
* Burp Suite is an effective tool for identifying and testing SQL Injection vulnerabilities in a controlled environment.
