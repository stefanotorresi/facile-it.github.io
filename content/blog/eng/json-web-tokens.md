---
authors: ["sandro"]
comments: true
date: "2016-04-02"
draft: true
image: "/images/cover.jpg"
menu: ""
share: true
categories: [English, JWT, Security]
title: "Json Web Token"

languageCode: "en-En"
type: "post"

---
![JWT](/images/json-web-tokens/logo.svg)

JSON Web Token (JWT) is an open standard [RFC 7519](https://tools.ietf.org/html/rfc7519) that defines a compact, self-contained and secure way for transmitting information between two parties. 


### Compact ?

Using javascript object notation to represent the data, means saving lots of bytes when the token goes over the network.
 
**Because of it's size** it can be sent over an URL or inside an HTTP Header and can be **easily parsed by a browser**.

This is more clear when comparing json to other standards like [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) that uses a very verbose XML structure.

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

The token is represented as three base64 encoded strings:

### Headers
The token type and the algorithm used to sign.

``` javascript
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
It's the data sent with the token. It contains metadata and information like expiration, audience, or subject and whatever you need.

``` javascript
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

All data transported is organized in claims, statements about an entity (typically, the user). There are three types of claims: reserved, public, and private claims.

- **Reserved claims**: a set of predefined claims, conceived to provide a set of useful information. Some of them are: iss (issuer), exp (expiration time), sub (subject), aud (audience), among others.
- **Public claims**: defined at will, but, in order to avoid collisions, they should be defined in the [IANA JSON Web Token Registry](http://www.iana.org/assignments/jwt/jwt.xhtml) or be described as a URI that contains a collision resistant namespace.
- **Private claims**: custom claims created to share information between parties that agree with using them.

### Sign
It is obtained from hashing headers and payload with a secret

``` javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

## Why use Json Web Token ?
I started my study about JWT because I needed to authenticate an API. I was searching for a smart method to authenticate a request, without querying the database each time, even if the token was invalid or expired.
Thanks to JWT self verification, I could discard every request in which the token was invalid (a fake token not signed by my application) or expired.
Beyond this specific use case, JWT can be useful also to securely transmit data to messagging applications.