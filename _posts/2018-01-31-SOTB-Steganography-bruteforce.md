---
layout: post
cover: True
title: "SOTB2018 Steganography BruteForce"
date:   2018-01-31 12:00:00
tags: CTF SOTB steganography
categories: 'CTF'
navigation: True
---
<br>
For this challenge, I did almost completed it, but I really did silly mistake. Overall, I like this challenge but I wasn't able to get full points! because the time is up for the competetion!
The real challenge is posted as the following image

<p align="center">
  <img src="/assets/ctf/steg_chall.jpg" alt="Steg challenge - SOTB" />
</p>

The prompt of the challenge said to brute force the image to get the secret!! so that I missed to run `strings` command on the image itself! stuiped me!!

```bash
root@kali:~/SOTB/remote/# strings steg_chall.jpg 
JFIF
$3br
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
	#3R
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
d~_\~}(
c<c?
...
...
...
G<}}G
9=q@
FN0@
;p=~^=9
=O?C
muduser@servername: Steghide's in plain sight... crack me
```

As obvious that last line gave a hint about the username which I missed `muduser`

Next, which was the first thing that I did actually is run `steghide` from Kali:

```bash
root@kali:~/SOTB/remote/# steghide extract -sf steg_chall.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
root@kali:~/SOTB/remote/# 
```

Huh? It asked for a password which confirmed that we need to do password. I found also on other challenges a dictionary words!! let's gave it try!

Using `for` loop in bash always works for me!

```bash
root@kali:~/SOTB/remote/# for i in $(cat /root/SOTB/sotb_dictionary.txt); do echo '[+] Trying ' $i; steghide extract -sf steg_chall.jpg --passphrase $i; done 
steghide: could not extract any data with that passphrase!
...
...
...
[+] Trying danielle
steghide: could not extract any data with that passphrase!
[+] Trying forever
steghide: could not extract any data with that passphrase!
[+] Trying dragon
wrote extracted data to "secrets.txt".
[+] Trying computer
steghide: could not extract any data with that passphrase!
[+] Trying whatever
steghide: could not extract any data with that passphrase!
[+] Trying family
...
..;
...
```
`dragon` was the password to get the `secret.txt` which looks like this:

```bash
team_server.shellontheborder 5035 
-----BEGIN RSA PRIVATE KEY-----
MIIJKgIBAAKCAgEAmxlDz34YckruO8rREY8bIh/b3d0KAs+MPa2PgIPLsM9EL2WF
qYEjhCDjy1OdtthELUPQGNTVOsHRVd1XCCe3qeWbiq98II6/TDgdv5zadKsJWJKC
sZvxALuqem5hny7O+GOQxmf33pIe7ogDv0x8YzYqQmiqRzG+sYtlqJ7a8TdXLck3
2jFaC3+k6ORyVQoXYTYfCtt2SpPQUAAJ998xzktBkERMm9Ap87XXnQOWZLR5Hi6j
BvhYl0KKHWsmQ9vtpPjrO9BgxZfl2qgn3yMxGCU/eEQy7sYYGnJ6lXgTfvilFjun
WUgB+YdOMXcBQqKNrsXIG2ZyPdod8pJPP0ajH7igfWzATZ3FOfDZFosd39ntAmdK
R+Q12f/0zOs4ZzbYk2wTwfMKbsHCc7JF7FXCqCGLlngZ6VlsChn8tcn/9fOZ4KKI
+jwrkPmdo/w5ksA5DLAAwa1U3g+8O6smSqixW37Qg/jo3u7ijUQ15+WzS6D3jzTI
root@kali:~/SOTB/remote/Steghide-Brute-Force-Tool/test# 
```
which showed that the server name and the port number and the most import thing is a private key for the user!!!

All components are there! so now just use ssh to login to the system! such

```bash
root@kali:~/SOTB/remote/# ssh team_server.shellontheborder.com -p 5035 -l muduser
```

this should let you connect to the server and read the flag :( and it propely should look like this:
`FLAG{I_SHOULDNT_MISS_AN_EASY_FLAG_SUCH_THIS}`

For me, I will always begin the challenge using `strings` command in CTF events! :P


PS: After talking to SOTB guys, they mentioned that there is still another puzzle!! Anyway I'm keeping this post as it is. it may help others in the future.

Yasir


