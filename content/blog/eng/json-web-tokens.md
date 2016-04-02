---
authors: ["sandro"]
comments: true
date: "2016-04-2"
draft: true
image: "/images/cover.jpg"
menu: ""
share: true
categories: [English, JWT, Security]
title: "JWT - Json Web Tokens"

languageCode: "en-En"
type: "post"

---
![JWT](/images/json-web-tokens/logo.svg)

JSON Web Token (JWT) is an open standard [RFC 7519](https://tools.ietf.org/html/rfc7519) that defines a compact, self-contained and secure way for transmitting informations between two parties. 


### Compact ?

Using javascript object notation to represent the data we want to transmit, results in saving lots of bytes when the token goes over the network.
 
**Because of it's size** it can be sent over an URL or inside an HTTP Header and can be **easily parsed by a browser**.

This is more clear when comparing json to other standard like [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) that uses a very verbose XML structure.

### Secure ?

JWT can be verified and trusted because it is digitally signed using a secret (usually with [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) algorithm) or a keypair with [RSA](https://en.wikipedia.org/wiki/RSA).

### Self-contained ?

The payload of the token contains all the required data to verify itself and, for example, transport the user data to avoid querying the database more than once.

## How the token presents itself

>eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
>
>.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
>
>.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

The toekn is representend as three base64 encoded string 

### Headers
Represents the token type and the algorithm used to sign.

``` javascript
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
It's the data sent with the token, can contain metadata and informations like expiration, audience, or subject and whatever you need.

``` javascript
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

All data transported is organized in claims, statements about an entity (typically, the user). There are three types of claims: reserved, public, and private claims.

- **Reserved claims**: a set of predefined claims, thought to provide a set of useful informations. Some of them are: iss (issuer), exp (expiration time), sub (subject), aud (audience), among others.
- **Public claims**: defined at will, but, to avoid collisions, they should be defined in the [IANA JSON Web Token Registry](http://www.iana.org/assignments/jwt/jwt.xhtml) or be defined as a URI that contains a collision resistant namespace.
- **Private claims**: custom claims created to share information between parties that agree on using them.

### Sign
Obtained from hashing headers and payload with a secret

``` javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

## Why use Json Web Token ?
I Started my study about JWT to authenticate an API. I was searching for a smart method to authenticate a request, whitout the needing of querying the database each time, even if the token where not valid or expired.
I found the solution in JWT, because of it's self verfication that helps me to discard every request where the token is not valid (a fake token not signed by my application) or expired.
But this is only my use case, JWT can be usefull to simply securely transmit data for example on a messagging application. Remeber only that all the data sent and received is not encrypted but only base64 encoded