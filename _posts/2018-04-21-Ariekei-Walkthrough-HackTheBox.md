---
layout: post
cover: True
title: "HackTheBox - Ariekei Walkthrough"
date:   2018-04-21 12:00:00
tags: CTF HackTheBox Hacking Docker 
categories: 'HackTheBox'
navigation: True
---
<br>
<p>In this article, I am going to walk you through the steps of how to hack `Ariekei` machine. `Ariekei` is one of the best machines that I have ever played. This machine has been rated as a hard box and it is really does. It is built based on Docker technology which means that it has many containers in it. This box also requires a pivoting skills till you reach the goal of pwning this machine which is reading the `root.txt` flag. The target host IP address is: `10.10.10.65`</p>

<p align="center">
  <img src="/assets/images/Ariekei/0-rank.png" />
</p>

<h2>Scanning the machine</h2>
As usual, starting scan the target host using `nmap` tool for `65535` ports for any running service. The following is the scan output:

<p align="center">
  <img src="/assets/images/Ariekei/1-nmap.png" />
</p>

Interesting, there are three ports open. Let examine port `443`. By visiting the target host ip address using a browser. For now, nothing important and in addition there is no web application is running. The showing message is `Maintenance - this site under development`. Next, checking the ssl for any well-known vulnerabilities using `sslscan` tool.
The tool didn’t show us any vulnerability but instead it shows some piece of important information about internal hostnames.

<p align="center">
  <img src="/assets/images/Ariekei/2-sslscan.png" />
</p>

For now, it is important to add those dns names into my `/etc/hosts` so that I can check them. Once this done, we can visit them by using the browser via https and see if this host is using `apache vhost` or not. Indeed it does. By visiting `calvin.ariekei.htb`, I’m getting another webpage which in this case `Not found page!`. It is ok, but this mean that it uses virtual host. 

So far, we have good starting point. But let finish up which enumerate the other 2 ports just identifying what they are at least. By using `nc` connect to those ports, we can see that the target is using `ssh` service on both ports.

<h2>Enumerating HTTPS</h2>
First, let scan `beehive.ariekei.htb` which seems same as `ariekei.htb` because it shows us the same page which is `the maintenance page`. Start scanning using `dirsearch` for looking any interesting files on the web server.

<p align="center">
  <img src="/assets/images/Ariekei/3-dirsearch.png" />
</p>

Interesting, the initial scan shows that this host has `blog` directory. Let also scan `blog` folder using `dirsearch` tool. By scanning it again, I have found `LICENSE` file which reveals the type of blog that the host is using. With some google about `Clean Blackrock` which mentioned in the previous file, I found that it is static blog - by the end of scanning, found `README.md` which confirms what I was thinking thus scanning this folder doesn’t give us any extra info. Using `cgi.txt` wordlist, I was able to find interesting folder and file `cgi-bin/stats`

<p align="center">
  <img src="/assets/images/Ariekei/4-cgi.png" />
</p>

It looks that some sort of script is running here. One of important vulnerabilities such this is “ShellShock” lets try it and see.

<p align="center">
  <img src="/assets/images/Ariekei/5-waf.png" />
</p>
Getting funny face which it seems to me there is WebApp firewall `waf` but keeps this in mind for later use!

<h2>Upload our shell/ImageTragick!</h2>
Now let scan the other host that we found which `calvin.ariekei.htb` if it has any interesting app or will find if there any `waf` there too. As soon start the scanning, I found `upload` folder. Aha, we have uploader, and the title is `Image converter` which first thing came to my mind is `ImageTragick` vulnerability. Lets give it try and see what is the output but first will use sleep command to make sure the vulnerability works or not. As soon upload `exploit.mvg`, we can see that the web server sleeps for 10 seconds which confirms that the vulnerability works.

<p align="center">
  <img src="/assets/images/Ariekei/6-ImageTragick.png" />
</p>

Lets try to upload our shell to the web-server and get reverse-shell. I am usually using python `python-pty-shells` which makes the life easier. My payload looks like the following

<p align="center">
  <img src="/assets/images/Ariekei/7-payload.png" />
</p>

And as result, we make the target host send request to our web server to grab my file.

<p align="center">
  <img src="/assets/images/Ariekei/8-webserver.png" />
</p>

<h2>Reverse Shell BUT!</h2>
By executing the uploaded using python, I was successfully received the shell back to me.

<p align="center">
  <img src="/assets/images/Ariekei/9-python-reverseshell.png" />
</p>

It is weird that we have a root access but by checking `/proc/1/cgroup`, it make sense since we are in the docker environment. After enumerating the system, I was able to find “/common” folder that has many useful information such root credentials

<p align="center">
  <img src="/assets/images/Ariekei/10-ssh access.png" />
</p>

<h2>SSH to port 1022</h2>
I also find a network topology for the entire network that is running on the host machine, and also found private and public keys under `.secret` folder. But the host doesn’t have any useful commands that helps me to go further. However, using that private key, I was able to login to the ssh on port 1022 and this time we logged in to another new docker container that has two interfaces which means connected to two networks (172.23.0.253 and 172.24.0.253).

<p align="center">
  <img src="/assets/images/Ariekei/11-sshaccess2.png" />
</p>

Again we see the seem folder for the docker environment which is `/common`. Now there is a `info.png` it seems to be interesting. Lets download it and check it out. At first, the picture didn’t work and I checked its header which showed that it is `Web/P image` that a new picture type that isn’t supported yet by many applications. Thus, I converted it from `Web/P image` to png using one of the online websites.

<p align="center">
  <img src="/assets/images/Ariekei/12-net-topology.png" />
</p>

Boom, we have gotten the network topology and it shows that we are logged into bastion-live docker machine which connects two networks. Now it also shows that `calvin and beehive` machines which are  behind a firewall as we expected. For now we need to reach `beehive` host from the internal network because it don’t have internal firewall, thus we still can use the `ShellShock` vulnerability that we tried from `bastion` machine. 

<h2>Local/Remote Port forward</h2>
However, there is a problem which is curl is not installed on the system and we don’t have internet access to install it. The idea that I got is now to do reverse shell to my Kali machine and do that internally from my machine.

<p align="center">
  <img src="/assets/images/Ariekei/13-remote-port-forward.png" />
</p>
We can also open local port 80 to forward to remote port 80 as following, both of them are working fine.
<p align="center">
  <img src="/assets/images/Ariekei/14-local-port-forward.png" />

</p>

<h2>ShellShock exploit behind the Firewall</h2>
I have successfully did remote port forward to my machine via port 8080 that forward to port 80 on host `172.24.0.2`. Then, tried to execute the curl command to exploit the application to print `Yas3r` which is my name.

<p align="center">
  <img src="/assets/images/Ariekei/15-shellshock-payload.png" />
</p>

And finally receiving a reverse shell by executing the following command:

<p align="center">
  <img src="/assets/images/Ariekei/16-shellshock-reverseshell.png" />
</p>

<h2>Getting user's flag</h2>
Previously under `/common` folder, we found a containers folder that has the root password. Let’s try it.

<p align="center">
  <img src="/assets/images/Ariekei/17-found-pass.png" />
</p>

Then, after getting root user on the system, I found private and public keys that related to user called `spanishdancer`. In addition, I found `user.txt` flag under same directory that belong to the user.

<p align="center">
  <img src="/assets/images/Ariekei/18-user-flag.png" />
</p>

Now, it is time to download those keys to connect to the main ssh server which listens on port 22. However, it requires a passphrase!

<p align="center">
  <img src="/assets/images/Ariekei/19-passphrase.png" />
</p>

The best way to crack this type of passphrase is by using `John the Ripper` tool, but first it requires to convert it using `ssh2john` to be compatible with `John`.

<p align="center">
  <img src="/assets/images/Ariekei/20-ssh2john.png" />
</p>

Now, we have the private key and the passphrase. We are ready to login to the main host machine that is running docker services. And here we go:

<p align="center">
  <img src="/assets/images/Ariekei/21-sshaccess.png" />
</p>

<h2>Getting root flag - Privilege escalation</h2>

By running `id` command, we can see that this user `spanishdancer` is part of docker group which means he can manage the entire docker environment.

<p align="center">
  <img src="/assets/images/Ariekei/22-docker.png" />
</p>

Now to read the host files as root including the root directory, if the host is sharing the file system which is vulnerability that allows the attacker to use the following command to mount the host files system including root owner.

<p align="center">
  <img src="/assets/images/Ariekei/23-docker-privesc.png" />
</p>

In order to get an actual root access, I can update the `/hostOS/etc/passwd` password and then change user using `su` to enter the new password.

<p align="center">
  <img src="/assets/images/Ariekei/24-root-access.png" />
</p>

It was a nice machine and learned a lot from it. Docker was a nice techn. that could be used in CTF and boot2root competitions. I hope you like this article.

<p align="center">
  <img src="/assets/images/Ariekei/badge.png" />
</p>

Regards,

`Yas3r`
