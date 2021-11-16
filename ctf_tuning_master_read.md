# CTF Tuning Master (Read) 暈暈陀陀 1 (150 point, 15 solves, ★☆☆☆☆)

Solved by: Kaiziron and grhkm

### Description :
```
Do you know how to cast spells, Shaman King?

http://偽女子.柳枊曳曳棘朿.ctf通靈師.hkcert.dsa.2048.pub:28237

http://偽女子.哪咧嘩唔咬哤.ctf通靈師.hkcert.dsa.2048.pub:28237

Part 1: Read the source code of home page
```
---
### Solution :

The url are using punycode
```
http://偽女子.柳枊曳曳棘朿.ctf通靈師.hkcert.dsa.2048.pub:28237
http://xn--czq51t5pb.xn--mova2urk5r76b.xn--ctf-tk2f171td9i.hkcert.dsa.2048.pub:28237/
```
```
http://偽女子.哪咧嘩唔咬哤.ctf通靈師.hkcert.dsa.2048.pub:28237
http://xn--czq51t5pb.xn--surk2ova5so5d.xn--ctf-tk2f171td9i.hkcert.dsa.2048.pub:28237/
```
First url :
![](https://i.imgur.com/Cl4T2LN.png)

It is giving out weird numbers

Second url :
![](https://i.imgur.com/ggc2hrX.png)

It is giving out some unprintable characters

Then I try to remove part of the domain name of first url

![](https://i.imgur.com/TnzCmpG.png)

The data its showing has changed

If I convert the hex `2f7661722f7777772f68746d6c2f696e6465782e706870` to ascii, it will become `/var/www/html/index.php` which is the full path of the file

Then, I do the same on the second url

![](https://i.imgur.com/DyAC8Ev.png)

it show error for hex2bin() function


Then I tried to remove a character on the first url :

![](https://i.imgur.com/AjkWYLQ.png)

```
Fatal error: Uncaught Error: Call to undefined function bin2h() in /var/www/html/index.php:1 Stack trace: #0 [internal function]: {closure}('/var/www/html/i...', 'xn--mova2urkt1d') #1 /var/www/html/index.php(1): array_reduce(Array, Object(Closure), '/var/www/html/i...') #2 {main} thrown in /var/www/html/index.php on line 1
```

Then it has error showing its trying to call the bin2h() function


Then I realize, is it related to the host http request header

The bin2h() function originally should be bin2hex(), but I removed some character, so it becomes bin2h()

```
curl -H 'Host: xn--czq51t5pb.xn--mova2urk5r76b.xn--ctf-tk2f171td9i.hkcert.dsa.2048.pub' http://xn--czq51t5pb.xn--mova2urk5r76b.xn--ctf-tk2f171td9i.hkcert.dsa.2048.pub:28237/
3734363665383864343539356638366230643534353466336439316435373639
curl -H 'Host: xn--czq51t5pb.xn--mova2urk5r76b.x.x.x.x.x' http://xn--czq51t5pb.xn--mova2urk5r76b.xn--ctf-tk2f171td9i.hkcert.dsa.2048.pub:28237/
3734363665383864343539356638366230643534353466336439316435373639
```


Then I found out that it is taking in the host header using `.` as the delimiter, it will ignore the top level domain to the fifth level domain part as modifying it won't affect the response of the page, and from the sixth level, it will take in as input 

This is the url that will show error for hex2bin() function
```
http://哪咧嘩唔咬哤.ctf通靈師.hkcert.dsa.2048.pub:28237
http://xn--surk2ova5so5d.xn--ctf-tk2f171td9i.hkcert.dsa.2048.pub:28237/
```

This part has the 2 character :
```xn--surk2ova5so5d```

and this part `urk2ova` can be converted to `hex2bin` by ROT13 

It will ignore the 5 characters on the left and 5 characters on the right, and execute the function using the full path of the file `/var/www/html/index.php` as the argument

The challenge is the read the file, the argument is the file path, so I can just use `show_source` to read the source

Convert `show_source` to `fubj_fbhepr` by ROT13

Then just set the http host header to 
```Host: xxxxxfubj_fbheprxxxxx.x.x.x.x.x```

![](https://i.imgur.com/XvbxH7s.png)

```php
<?=array_reduce(explode(".",$_SERVER["HTTP_HOST"],-5),function($p,$q){return str_rot13(substr($q,5,-5))($p);},__FILE__);//hkcert21{HA*HA*HA*NA*HA*NA*SE*HA*HA*NA*NA*SE}
```
### Flag :
`hkcert21{HA*HA*HA*NA*HA*NA*SE*HA*HA*NA*NA*SE}`