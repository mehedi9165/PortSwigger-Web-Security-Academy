# 🛡️ PortSwigger Web Security Academy

Welcome to my PortSwigger Web Security Academy repository.

This repository documents my hands-on practice while completing the PortSwigger Web Security Academy labs. Every write-up includes my methodology, payloads, screenshots, and mitigation techniques.

---

# 👨‍💻 About Me

**Mehedi Hasan Rakib**

- Cybersecurity Enthusiast
- Ethical Hacker
- Digital Forensics Learner
- Web Application Penetration Tester

---

# 🎯 Objectives

- Complete all PortSwigger Web Security Academy Labs
- Improve practical Web Penetration Testing skills
- Build a professional GitHub portfolio
- Prepare for PNPT and real-world penetration testing engagements

---

# 📚 Lab Categories

| Category | Status |
|----------|--------|
| SQL Injection | ⏳ Completed |
| Cross-Site Scripting (XSS) | ⏳ In Progress |
| Authentication | ⏳ |
| Access Control | ⏳ |
| CSRF | ⏳ |
| SSRF | ⏳ |
| XXE | ⏳ |
| File Upload | ⏳ |
| Path Traversal | ⏳ |
| OS Command Injection | ⏳ |
| JWT | ⏳ |
| WebSockets | ⏳ |
| GraphQL | ⏳ |
| Race Conditions | ⏳ |
| API Testing | ⏳ |

---

# 📂 Repository Structure

```
PortSwigger-Web-Security-Academy

├── SQL-Injection
├── XSS
├── Authentication
├── Access-Control
├── CSRF
├── SSRF
├── XXE
├── File-Upload
├── Path-Traversal
├── JWT
├── GraphQL
├── WebSockets
├── API-Testing
└── README.md
```

---

# 📝 Each Lab Contains

- Methodology
- Payloads Used
- Burp Suite Requests
- Screenshots

---

# 🛠️ Tools Used

- Burp Suite Professional/Community
- Firefox
- Google Chrome
- Kali Linux
- GitHub

---

# 📈 Progress

- Total Labs Completed: 15
- SQL Injection: 15
- XSS: 0
- Authentication: 0

---

## ⭐ Thank you for visiting my repository.
```

# Mitigation
- Use parameterized queries (prepared statements) instead of concatenating user input into SQL queries.
- Validate user input using allow-list validation and reject unexpected values.
- Apply the principle of least privilege to database accounts to minimize the impact of successful attacks.
- Return generic error messages to users while logging detailed errors securely on the server.
- Avoid constructing dynamic SQL queries with untrusted input, including within stored procedures.
- Keep the application, database, framework, and dependencies updated with security patches.
- Deploy a Web Application Firewall (WAF) as an additional layer of protection against common SQL injection attacks.
- Perform regular secure code reviews, vulnerability assessments, and penetration testing to identify and remediate SQL injection vulnerabilities.
