---
authors: ["sandro"]
comments: true
date: "2016-04-05"
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

When I started my study about JWT, I was searching for a smart method to authenticate a request, without querying the database each time to check the applicant reliability.
I needed a token or something similar with the ability to validate itself and flexible enough to customize the validation strategy.
Immagine for example a web application where a user can login and obtain a "pass" with his name and an expiration time, and this pass will let him ask for resources until the pass expires, and only if the issuer is trusted.
Thanks to JWT self verification, I could discard every request where the token is invalid (a fake token not signed by my application) or expired.
Beyond this specific use case, JWT can be also useful to securely transmit data to other applications.

Now let me introduce the standard: JSON Web Token (JWT) is an open standard [RFC 7519](https://tools.ietf.org/html/rfc7519) that defines a compact, self-contained and secure way for transmitting information between two parties. 

Using javascript object notation to represent the data means two things: 

- saving lots of bytes when the token goes over the network, **because of it's size** it can be sent over an URL or inside an HTTP Header
- it can be **easily parsed by a browser** and consumed by a client application

This is more clear when comparing json to other standards like [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) that uses a very verbose XML structure.

A JWT token can be trusted because it is digitally signed using a secret (usually with [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) algorithm) or a keypair with [RSA](https://en.wikipedia.org/wiki/RSA).
Its payload contains all the required data to verify itself and, for example, transport the user data to avoid querying the database more than once.

## How the token presents itself
A token is represented as three base64 encoded strings joined by two points (here represented on three lines because of layout problems).

``` javascript
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

The first part of the string contains the headers, "alg" is the algorithm used to secure the token and "typ" is the type; in this case as we can see that I used an HMAC SHA256 to sign the JWT token

``` javascript
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
It's the data sent with the token in the second string. It contains metadata and information like expiration, audience, or subject and whatever you need.

``` javascript
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

All data transported is organized in **claims**, statements about an entity (typically, the user). There are three types of claims: reserved, public, and private claims.

- **Reserved claims**: a set of predefined claims, conceived to provide a set of useful information. Some of them are: `iss` (issuer), `exp` (expiration time), `sub` (subject), `aud` (audience), among others.
- **Public claims**: defined at will but, in order to avoid collisions, they should be defined in the [IANA JSON Web Token Registry](http://www.iana.org/assignments/jwt/jwt.xhtml) or be described as a URI that contains a collision resistant namespace.
- **Private claims**: custom claims created to share information between parties that agree with using them.

### Sign
The third part of the string is the sign, obtained from hashing headers and payload with a secret using the algorithm described in headers.

``` javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

As said before the more interesting feature of JWT is in it's flexibility. It can be created with the claims you need, carry your data and validated on other claims (standard or not).
As an example, if on authentication I want to be sure that the token is issued by my application and that it is not older than 1 hour I can generate a token with a payload like this:

``` javascript
{
  "iss": "1234567890", // my application code
  "exp": "1459868400", // 2016-04-04 15:00:00 Expiration time
  "uid": 159, // the user id
  "name": "Alessandro Galli"
}
```

Once my application receives a request with this token, an authenticator component, will check the two claims (`iss`, `exp`) to be sure of the assertions made before, and then validate the sign. Furthermore, with the additional claims it can login the user without querying the database and complete the request.

As a solid standard, JWT has been adopted by a large number of users and it has libraries for almost every programming language: PHP, Java, Go, Python, Javascript, Ruby, Elixir, Scala, .Net.

You can find a full list of available and trusted libraries on [JWT.io](https://jwt.io/#libraries-io)

In the next weeks, I will write a post and release my personal authentication example, using PHP with the Symfony framework.
