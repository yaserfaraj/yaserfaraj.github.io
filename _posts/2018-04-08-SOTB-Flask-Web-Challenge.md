---
layout: post
cover: True
title: "SOTB - Flask Web Challenge "
date:   2018-04-08 12:00:00
tags: CTF SOTB ITsecurity Web-Hacking
categories: 'CTF'
navigation: True
---
<br>
Hi everyone. We had a lot of fun at #sp4rkcon this weekend, and it was amazing as last year. I also met great people including `@earcmile` from SOTB. He told me that he re-published couple of web challenges from the `Shell On The Border` capture the flag. Thus, I re-did the challange and I would like to share it with you'all. 

You can find the challenge on: `http://ctf.fs2600.net:5013/`

As usual, I started with scanning the port to get what we are dealing with as following:

<p align="center">
  <img src="/assets/images/flask/1-nmap.png" />
</p>

As obvious that we are dealing with `gunicorn flask 19.7.1` which is python framwork. By sending a `GET` request using `curl`, I can confirm the web app is up and running and showed the hint of the get parameter which is `name`!

<p align="center">
  <img src="/assets/images/flask/2-first-curl.png" />
</p>

By doing some research (at that time), I found couple websites that gives a great details about exploting such a framework. References are attached on the end of this post.

After many attempts, I was able to confirm this app has a `RCE` vulnerablity.

<p align="center">
  <img src="/assets/images/flask/3-rce.png" />
</p>

It is obvious that the application returns the mathmatical operation of `1x2` and `2x4`

By going through one of the reference, I was able to list `classes` that the framwork use:

<p align="center">
  <img src="/assets/images/flask/4-list-classes.png" />
</p>

As we can see that there are two classes. The first one is `string` and the other one is `objects`. The application lists the elemsts as an array which means str has index `0` and the objects class has index of `1`

<p align="center">
  <img src="/assets/images/flask/5-list-subclasses.png" />
</p>

By listing all sub-classes of the object main class, we can see that it has many of functions (more than 250). Next, it is the time to find an interesting function that cause a `RCE` and if you are familier with `python` then the answer is `Popen`
Using the find `Ctrl + F` to search for it and inded, it is there. But the challenge how to know the index location of the function (we have more than 250 elements in this array)? 

To proof I am on the right track. I viewed the first and second elements as following:

<p align="center">
  <img src="/assets/images/flask/6-enumarate-array-position.png" />
</p>

<p align="center">
  <img src="/assets/images/flask/7-enumarate-array-position2.png" />
</p>

For me, I run a bash script that goes through 300 loop to list the function name and grep the `Popen` function and print its index location.

<p align="center">
  <img src="/assets/images/flask/9-bash-for-loop-automate-to-find-popen-fun.png" />
</p>

Great. We got the index location which is `228` and it is time to use this function to wirte a new file into /tmp to get the flag.

<p align="center">
  <img src="/assets/images/flask/10-got-flag-using-cmd.png" />
</p>

And here we got it :)

Thanks for reading this article and I hope you like it.

[1] ref: https://nvisium.com/resources/blog/2015/12/07/injecting-flask.html<br>
[2] ref: https://nvisium.com/resources/blog/2016/03/09/exploring-ssti-in-flask-jinja2.html

Regards,

Yas3r


