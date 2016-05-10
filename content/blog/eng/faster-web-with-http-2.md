---
authors: ["sergio"]
comments: true
date: "2016-05-08"
draft: true
image: ""
menu: ""
share: true
categories: [English, HTTP, Networking]
title: "Faster Web with HTTP/2"
languageCode: "en-US"
type: "post"
---

HTTP is the protocol that powers the Web. It was originally designed in 1996 for transferring and manipulating simple text-based documents (mainly hypertext resources).
Nowadays it's been adopted for many different purposes. It's used for: multimedia content transfer, rich real-time session-based web applications, API messages dispatch, Internet of Things and much more.
For this reason, the HTTP specifications are continuously updated, by adding new features and improving performance.

In May 2015, the latest version 2.0 was standardized with *[RFC 7540](https://tools.ietf.org/html/rfc7540)*.
This introduced major differences on how the low-level protocol works.
From the application perspective, very little has changed: requests, responses, resources, headers and HTTP methods are still there.
New features have been added such as the possibility to **push resources to the client**.

The main reason that led to HTTP/2 is **improving performance**.
Previous versions of HTTP had critical problems that were not addressable without changing the low-level communication mechanisms.

## HTTP/1.1 issues
HTTP/2 deals with many of the performance issues of HTTP/1.1:

* HTTP/1.1 doesn't support headers compression even though it would be particularly effective against text-based data;
* HTTP/1.1 [request pipelining](https://en.wikipedia.org/wiki/HTTP_pipelining) is not so efficient due to [head-of-line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking);
* Modern web applications have complex resources that are linked to each other. HTTP/1.1 servers have no way to send all the related resources at once in order to avoid additional round-trips;
* HTTP/1.1 connections are designed to have short lifespans. Unfortunately, web resources change over time and additional connections are required to fetch the updated data.

People have adopted several tricks to mitigate these problems:

* Session data is usually persisted on the server to minimize the amount of headers sent at every request;
* To reduce latency and to achieve request multiplexing, browsers open several TCP connections during page load. This is not recommended: it consumes many more resources (both client-side and server-side) and [TCP congestion avoidance](https://en.wikipedia.org/wiki/TCP_congestion_control) techniques are no longer effective;
* Recent web applications prefer leaving connections open to reduce connection setup latency for subsequent requests;

## What's changed
HTTP/2 is **no longer text-based**; this means debugging could be somewhat difficult without using special tools.

Server and client communicate by sending small binary messages named **Frames**.
There are different types of frames. The most important ones are `DATA`, `HEADERS`, `SETTINGS` and `PUSH_PROMISE`.

HTTP/2 uses the concept of **streams**. Each stream is identified by an ID and it's allocated for every request/response lifecycle.
Several streams can be active at the same time on the same connection: this provides **real request pipelining and multiplexing**.

Each frame contains the ID of the stream they refer to. Thus, frames of different streams can be interleaved without interfering.

Servers, by sending the `PUSH_PROMISE` frame, can allocate new streams without waiting for an explicit request from the client. By the virtue of this, **the server can push resources to the client** and reduce the number of round-trip messages needed.

Http headers (both requests' and responses' ones) are always compressed and they are sent using one or more `HEADERS` frames.

*[Response Status-Line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec6.html#sec6.1)* and *[request Request-Line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1)* have been moved. They are now stored in the header section via special headers (pseudo-headers): `:method`, `:path`, `:status`, etc..

*Response Status-Line Reason-Phrase* has been completely removed: there is no equivalent in HTTP/2.

Servers and clients may use `PING` frames to check the underlying connection's state and to keep the connection alive even for a long period of time.

## Support by servers and user agents
HTTP/2 is actually a fork of [SPDY](http://dev.chromium.org/spdy/spdy-whitepaper), a protocol developed by Google.
A lot of different servers and clients already supported the SPDY protocol, so the transition from SPDY to HTTP/2 was pretty straightforward.

HTTP/2 was officially released in May 2015; since then **all major web servers have added support for HTTP/2** such as Apache HTTP, Apache Tomcat, Nginx, IIS and Jetty.

**All major browsers now support HTTP/2**, e.g. Firefox, Chrome, IE and Edge.

A the moment, they only allow HTTP/2 over TLS via [ALPN](https://tools.ietf.org/html/rfc7301) or [NPN](https://tools.ietf.org/html/draft-agl-tls-nextprotoneg-04) as Protocol Negotiation mechanisms.
RFC 7540 considers the case of HTTP/2 being used on cleartext connections as well, but actually this is not extensively used.

## Additional resources
The website [Http2demo.io](http://www.http2demo.io/) compares performance metrics between HTTP/1.1 and HTTP/2.

If you want to learn more about HTTP/2, you can read the official specifications (**[RFC 7540](https://tools.ietf.org/html/rfc7540)**) or download the book **[Http2 explained](https://daniel.haxx.se/http2/)** which also examines the reasons behind HTTP/2.
