# HKCERT CTF 2021 Jack Botnet Service Writeup

## Jack Botnet Service 1 (300 point, 12 solves, ★★★☆☆)
---
Solved by: Kaiziron and grhkm

### Description :
```
Jack is testing his HTTP botnet, what could go wrong?

Attachments
jack-botnet-service-1_064a2bd6bb9036f6e8788d8418e18300.zip
```
---
### Solution :

There is a pcap file in the zip file 
![](https://i.imgur.com/QebAqNY.png)

The challenge description shows that this challenge is about http, so I export all http object in the pcap using wireshark

![](https://i.imgur.com/pIEzo2o.png)

Then I just look at it manually 

```
cat 'watch%3fv=6omHDfHITZ4(10)' 
v=XUV863a1Lok&debug=eyJHVUlEIjoiMmFhOWNiNjljYSIsIlR5cGUiOjAsIk1ldGEiOiJhMDE4NTdiNThiIiwiSVYiOiJmSFk3N1VhdnhjeEZNTkg3M1V2S2JBPT0iLCJFbmNyeXB0ZWRNZXNzYWdlIjoiM2FPTU1sSVBuRmZHWDNhOXNoUjU1cDVhNE9JRU05dWd6RTZFVDg1UXRPb0F1V0V2eGFrTVp4TUUyaWxSRlVkTSIsIkhNQUMiOiJsS01KaWxRYW1TQzRPQVlPZ2JLc1AwWkFLWTdmVmZrQ2ZRNWVHUWRLT2JNPSIsIktleSI6IlZ1SHBQcTBhV2R1NXJySkxZMmhjR3BHS3BGdUJHaWdSYkRMNS9QZ0JwVDg9In0=
```

The data in debug parameter is base64 encoded
```
echo "eyJHVUlEIjoiMmFhOWNiNjljYSIsIlR5cGUiOjAsIk1ldGEiOiJhMDE4NTdiNThiIiwiSVYiOiJmSFk3N1VhdnhjeEZNTkg3M1V2S2JBPT0iLCJFbmNyeXB0ZWRNZXNzYWdlIjoiM2FPTU1sSVBuRmZHWDNhOXNoUjU1cDVhNE9JRU05dWd6RTZFVDg1UXRPb0F1V0V2eGFrTVp4TUUyaWxSRlVkTSIsIkhNQUMiOiJsS01KaWxRYW1TQzRPQVlPZ2JLc1AwWkFLWTdmVmZrQ2ZRNWVHUWRLT2JNPSIsIktleSI6IlZ1SHBQcTBhV2R1NXJySkxZMmhjR3BHS3BGdUJHaWdSYkRMNS9QZ0JwVDg9In0=" | base64 -d
{"GUID":"2aa9cb69ca","Type":0,"Meta":"a01857b58b","IV":"fHY77UavxcxFMNH73UvKbA==","EncryptedMessage":"3aOMMlIPnFfGX3a9shR55p5a4OIEM9ugzE6ET85QtOoAuWEvxakMZxME2ilRFUdM","HMAC":"lKMJilQamSC4OAYOgbKsP0ZAKY7fVfkCfQ5eGQdKObM=","Key":"VuHpPq0aWdu5rrJLY2hcGpGKpFuBGigRbDL5/PgBpT8="}
```

It contains AES encrypted data and the key, however not all http traffic has the key, but all traffic that has the key has the same key

Then I just automate the process of base64 decoding for all exported http objects :

```python
import os
import base64

files = os.listdir("./http")

for filename in files :
	filename = f"./http/{filename}"
	content = open(filename, "r").read()
	try :
		print(base64.b64decode(content.split('debug')[1][1:]))
		with open("data.txt", "ab") as f:
			f.write(base64.b64decode(content.split('debug')[1][1:]) + b"\n")
	except :
		print("cant find 'debug'")

```
After its exported, decrypt the AES encrypted data :

```python
from Crypto.Cipher import AES
from base64 import b64decode as bd


def decrypt(key, iv, msg):
    return AES.new(bd(key), AES.MODE_CBC, bd(iv)).decrypt(bd(msg))


key = "VuHpPq0aWdu5rrJLY2hcGpGKpFuBGigRbDL5/PgBpT8="
file = open('data.txt', 'r').readlines()
log = open('decrypted.txt', 'w')

for line in file:
    dt = eval(line)
    if "IV" not in dt or "EncryptedMessage" not in dt:
        continue
    print(f"mode={dt['Type']}")
    res = decrypt(key, dt["IV"], dt["EncryptedMessage"])
    try:
        res = res.decode()
    except:
        print("cannot decode", res)
    log.write(f'{res}\n')
```

The flag is in the decrypted data :
```
cat decrypted.txt | grep hkcert21{
{"status":"3","output":"TODO:\r\n1. Buy a Powerful Server\r\n2. Shoot Payload Everywhere\r\n3. Profit!!!\r\n\r\nhkcert21{1_am_t3ch_5avvy}\r\n\r\n\r\n"}		
```

### Flag :
`hkcert21{1_am_t3ch_5avvy}`

---


## Jack Botnet Service 2 (300 point, 8 solves, ★★★☆☆)
---
Solved by: Kaiziron

### Description :
```
Seems Jack is hiding a flag on his ture homepage! Can you find it out?

Note: There is no dependency between part 1 and part 2 (i.e. it is not necessary to get part 1 solved before solving part 2)
Attachments
jack-botnet-service-2_064a2bd6bb9036f6e8788d8418e18300.zip
```

---
### Solution :
(The zip file is the same as part 1)

In the pcap file, there is some http traffic
![](https://i.imgur.com/cbL4qoe.png)

The domain name of the server is seen in the host header :
http://d26dazakrob6tg.cloudfront.net/

![](https://i.imgur.com/CyRtrHq.png)

No matter what code I'm submitting, it will redirect to a youtube video, even I do the same request as the http traffic in the pcap file, I can't get the same response

So I decided to use `curl` 

```
curl -i http://d26dazakrob6tg.cloudfront.net/anything
HTTP/1.1 302 Found
Content-Type: text/html; charset=iso-8859-1
Content-Length: 294
Connection: keep-alive
Date: Sun, 14 Nov 2021 15:02:19 GMT
Server: Apache
Location: https://www.youtube.com/watch?v=XUV863a1Lok
X-Cache: Miss from cloudfront
Via: 1.1 53b2bbb13e5db590d598ee4e9aa9bd80.cloudfront.net (CloudFront)
X-Amz-Cf-Pop: HKG62-C2
X-Amz-Cf-Id: 4eGYpftW4Q6tnJb08h1B-U6QLklrFCBhjnMhcBmgOFtSbHsTlHMAyA==

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="https://www.youtube.com/watch?v=XUV863a1Lok">here</a>.</p>
<hr>
<address>Apache Server at vvideo.cc1emon.ml Port 80</address>
</body></html>
```
I found that all invalid path will has a 302 status code and redirect to a youtube video

What's strange about it is the Apache message disclosed a different domain name of the server

http://vvideo.cc1emon.ml/

![](https://i.imgur.com/OAXn9xX.png)

It is just the same page, however when I try to remove the `vvideo` subdomain, a new page is shown with the flag :

http://cc1emon.ml/

![](https://i.imgur.com/ZJrfbwh.jpg)

### Flag :
`hkcert21{00ps_th1s_p4g3_1s_n0t_y3t_r34dy}`

---
