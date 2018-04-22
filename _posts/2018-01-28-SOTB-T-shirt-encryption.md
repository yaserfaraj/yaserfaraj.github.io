---
layout: post
cover: True
title: "SOTB2018 T-Shirt challenge"
date:   2018-01-28 12:00:00
tags: CTF SOTB ITsecurity Encryption
categories: 'CTF'
navigation: True
---
Hi, It was a nice weekend at Fort Smith, Arkansas participating professional CTF `Shell On The Border`. It was an amazing event which has more 21 teams (4-players). First day, it was just a registration and orientation day which they were explaining the comptention's rule and also they distribtued some other stuff including interesting T-shirt! that has something on it!

<p align="center">
  <img src="/assets/images/tshirt.png" alt="T-shirt challenge - SOTB" />
</p>

Discussed this with my team-mate about the string which before even started this challenge. We figured out that the string is `Hex`, and it looks like encrypted and the last string some sort looks like key `SOTB2018`.

It took me an hour to solve this out because I have came across same encryption before.

I was thinking of `XOR` encryption but I need to confirm that before go any further! what I did is I know that the flag starts with `FLAG` string, so it is good idea to encrypt this using `XOR` and `SOTB2018` key.


```bash
root@yas3r:~#                                                # Encrypt stage
root@yas3r:~# echo -ne 'FLAG' | xxd -a                       #convert TEXT to HEX
00000000: 464c 4147                                FLAG
root@yas3r:~# echo -ne 'SOTB' | xxd -a
00000000: 534f 5442                                SOTB      #convert TEXT to HEX
root@yas3r:~# python -c "print hex(0x464c4147 ^ 0x534f5442)" #HEX XOR
0x15031505 		# compared it with the picture!! :)
root@yas3r:~#
root@yas3r:~# echo '0x15031505' | xxd -r -p                 #convert HEX back to TEXT
root@yas3r:~#
root@yas3r:~#
root@yas3r:~# python -c "print hex(0x15031505 ^ 0x534f5442)" #Decrypt stage
0x464c4147
root@yas3r:~# echo '0x464c4147' | xxd -r -p
FLAG
```

Now it's confirmed. It is time to do a python to do the magic. But there is something important here. The `key` has to be as same as the encrypt length! which means we need to repeate the `key` to be equal the encrypted message!


```python
root@yas3r:~/SOTB/encryption# cat ../t-shirt.py
enc = "\x15\x03\x15\x05\x49\x79\x6e\x50\x32\x2c\x3f\x27\x56\x6f\x45\x50\x36\x10\x13\x2b\x50\x43\x5e\x56\x0c\x2e\x3a\x26\x6d\x51\x5d\x54\x0c\x06\x0b\x25\x5d\x44\x6e\x4f\x32\x3c\x0b\x36\x5a\x59\x42\x67\x3f\x20\x21\x31\x4b\x6f\x45\x15\x20\x27\x3d\x30\x46\x11\x4c"

key = "\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31\x38\x53\x4f\x54\x42\x32\x30\x31"


decrypt = str()

for i in range(0, len(enc)):
  decrypt += chr(ord(key[i]) ^ ord(enc[i]))

print "plaintext is: {0}".format(decrypt)
```

AND BOOM!!

```bash
root@yas3r:~# python t-shirt.py
plaintext is: FLAG{I_hacked_the_Gibson_and_all_I_got_was_this_lousy_t-shirt!}
root@yas3r:~#
```

Yas3r


