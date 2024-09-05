---
title: PortSwigger - Web Cache Poisoning
date: '2024-08-23'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---
Caching in web applications involves storing copies of web resources (like HTML pages, images, and API responses) closer to the client to reduce latency, bandwidth usage, and server load. Here are some common types of web caches and how they work with HTTP examples:

### Types of Web Caches

1. **Browser Cache**: Stores resources on the user's device.
2. **Proxy Cache**: A shared cache that serves multiple users (e.g., corporate networks).
3. **Content Delivery Network (CDN) Cache**: Distributed cache servers located closer to users.

### How Caching Works

1. **HTTP Headers**: Caching behavior is controlled through HTTP headers. Here are some important headers:

    - **`Cache-Control`**: Directs how, and for how long, the resource should be cached.
    - **`Expires`**: Specifies the date/time after which the response is considered stale.
    - **`ETag`**: A unique identifier for a resource version, used for cache validation.
    - **`Last-Modified`**: The date/time when the resource was last modified, also used for cache validation.

### HTTP Caching Examples

1. **Cache-Control Header**:

    - `Cache-Control: no-cache`: The resource must be revalidated with the server before being served.
    - `Cache-Control: no-store`: The resource must not be cached.
    - `Cache-Control: max-age=3600`: The resource can be cached and considered fresh for 3600 seconds (1 hour).

  ```http
    GET /resource HTTP/1.1
    Host: example.com
    
    HTTP/1.1 200 OK
    Cache-Control: max-age=3600
```

2. **Expires Header**:

    - Specifies an absolute date/time when the resource becomes stale. Less flexible than `Cache-Control: max-age`.

```http
    GET /resource HTTP/1.1
    Host: example.com
    
    HTTP/1.1 200 OK
    Expires: Wed, 21 Oct 2023 07:28:00 GMT
```

3. **ETag and If-None-Match**:

    - ETag is a unique identifier for the resource version. The client can use `If-None-Match` to ask the server if the resource has changed.

```http
    GET /resource HTTP/1.1
    Host: example.com
    
    HTTP/1.1 200 OK
    ETag: "abc123"
```

- On subsequent requests, the client can use the ETag to check if the resource has changed.

```http
    GET /resource HTTP/1.1
    Host: example.com
    If-None-Match: "abc123"
    
    HTTP/1.1 304 Not Modified
```

4. **Last-Modified and If-Modified-Since**:

    - Last-Modified specifies the date/time the resource was last modified. The client can use `If-Modified-Since` to check if the resource has been modified since the last request.

```http
    GET /resource HTTP/1.1
    Host: example.com
    
    HTTP/1.1 200 OK
    Last-Modified: Wed, 21 Oct 2023 07:28:00 GMT
```

- On subsequent requests, the client can check if the resource has been modified.

```http
    GET /resource HTTP/1.1
    Host: example.com
    If-Modified-Since: Wed, 21 Oct 2023 07:28:00 GMT
    
    HTTP/1.1 304 Not Modified
```

### Example Scenario

Consider a simple scenario where a user visits a webpage, and the browser cache is used to store images and stylesheets:

1. **Initial Request**:
    - User requests `index.html`.
    - Server responds with `index.html` and includes `Cache-Control: max-age=3600` for static resources like images and stylesheets.

```http
    GET /index.html HTTP/1.1
    Host: example.com
    
    HTTP/1.1 200 OK
    Cache-Control: max-age=3600
```

2. **Subsequent Request**:
    - Within the next hour, if the user revisits the page, the browser will use the cached version of images and stylesheets without sending a request to the server.

```http
    GET /style.css HTTP/1.1
    Host: example.com
    
    (cached response used)
```

By utilizing caching headers effectively, web applications can significantly improve performance and reduce load on the server.


----


