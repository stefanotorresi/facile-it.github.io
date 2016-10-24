---
authors: ["salvatore-cordiano"]
date: "2016-10-24"
draft: false
share: true
categories: [English, MessagePack, JSON]
title: "MessagePack: an alternative to JSON?"
type: "post"
languageCode: "en-EN"
twitterImage: '/images/message-pack-an-alternative-to-json/share.png'
---
When we talk about data interchange in web applications, **[JSON](http://www.json.org/)** is the de-facto standard, especially in developing a RESTful web services. JSON won against its antagonist [XML](https://www.w3.org/XML/) (SOAP) without a battle, but it didn't prevent the development of alternatives like [Google’s **Protocol Buffers**](https://developers.google.com/protocol-buffers/), [Apache **Avro**](https://avro.apache.org/) or **[MessagePack](http://msgpack.org/)**. In being thorough, we should also mention [gzip](http://www.gzip.org/) JSON compression (sometimes also called "*JSONC*"), and [BSON](http://bsonspec.org/), a bin­ary-en­coded seri­al­iz­a­tion of JSON-like doc­u­ments, both derived directly from JSON. In this article we'll discuss MessagePack in depth.

![MessagePack](/images/message-pack-an-alternative-to-json/msgpack.png)

# What is MessagePack?{#what-is-messagepack}

« *MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller* ». To start using MessagePack we need to convert our application objects into MessagePack format: this process is called *serialization*, while the reverse process is called *deserialization*. The following example can help us better understand what we're talking about. Consider this simple JSON:

```json
{
	"id": 4,                    // integer
	"isActive": true,           // boolean
	"fullname": "Homer Simpson" // string
}
```

JSON requires 56 bytes to represent a very simple user object, while MessagePack only needs 38 bytes (compression ratio 1.47, yielding a 32% saving in size). See below the output of the serialization process for the above JSON:

```
83 a2 69 64 04 a8 69 73 41 63 74 69 76 65 c3 a8 66 75 6c 6c 6e 61 6d 65 ad 48 6f 6d 65 72 20 53 69 6d 70 73 6f 6e
```

We can see how MessagePack serialization works by reading the [official specification](https://github.com/msgpack/msgpack/blob/master/spec.md). Also, we can split the previous hexadecimal representation to emphasize and explain data types as follows:

```sql
83                                          // 3-element map
a2 69 64                                    // 2-byte string "id"
04                                          // integer 4
a8 69 73 41 63 74 69 76 65                  // 8-byte string "isActive" 
c3                                          // boolean true
a8 66 75 6c 6c 6e 61 6d 65                  // 8-byte string "fullname"
ad 48 6f 6d 65 72 20 53 69 6d 70 73 6f 6e   // 13-byte string "Homer Simpson"
                                            // total 38 bytes
```

Now it's very simple to figure out the meaning of the sentence « *Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves* » reported in the headline of MessagePack website.

The main features of MessagePack are:

- it's designed for network communication and to be transparently converted from and to JSON;
- it supports in-place updating, so it's possible to modify part of a stored object without reserializing it as a whole;
- it has a flexible [Remote Procedure Call (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call) and streaming API implementation;
- it supports [static-type-checking](https://en.wikipedia.org/wiki/Type_system#Static_type_checking).

# Supported data types{#supported-data-types}

Data types listed by the specification are very similar to those in JSON, that is:

* **Integer** represents an `integer`;
* **Boolean** represents `true` or `false`;
* **Nil** represents `nil`;
* **Float** represents a [IEEE 754](https://en.wikipedia.org/wiki/IEEE_floating_point) double precision floating point numbers including `NaN` and `Infinity`;
* **String** is a `raw type` and it represents a UTF-8 string;
* **Binary** is a `raw type` and it represents a binary data using byte array;
* **Array** represents a sequence of objects;
* **Map** represents a dictionary (key-value pairs of objects);
* **Extension** represents a tuple of data whose meaning is defined by applications.

# A naive benchmark

Up to this point our reasoning was focused on **space efficiency**, but a good theoretical computing scientist would have criticized us for we didn't mention **time complexity**. In fact, the process of data *compression* and *decompression* is not negligible. We can analyze and compare, for example, the time required to parse a JSON document and to unpack a MessagePack document: that's not completely scientific, but it's a start.

We wrote two **[Node.js](https://nodejs.org/en/)** scripts to execute 1 million JSON parsing and 1 million MessagePack unpacking operations of a [sample document](https://github.com/salvatorecordiano/facile-it-message-pack-benchmark/blob/master/document.json) containing the same data in the two formats.

A simplified version of the code could be something like this:

```javascript
// inside script "test_parse_json.js"
for (var i = 0;i<1000000;i++) {
    JSON.parse(jsonDocument); // JSON document parsing
}

// inside script "test_unpack_msgpack.js"
for (var i = 0;i<1000000;i++) {
    msgpack.unpack(msgPackDocument); // MessagePack document unpacking
}
```

To easily profile our scripts we can run them as below:

```sql
aiace:msgpack parallel$ time node test_parse_json.js

real	0m47.296s
user	0m47.202s
sys	0m0.059s

aiace:msgpack parallel$ time node test_unpack_msgpack.js

real	1m47.244s
user	1m47.050s
sys	0m0.120s
```

Numbers are self-explanatory: the MessagePack binary is *smaller* than the minified JSON, but MessagePack deserialization is clearly *slower* than JSON parsing process.

Before going on, we also need to say that all tests are executed in the following described environment and the full code of this benchmark is free available [here](https://github.com/salvatorecordiano/facile-it-message-pack-benchmark/).

```bash
// Machine
OS : Darwin 15.6 (x64)
RAM: 16.384 MB
CPU: 2.200 MHz Intel(R) Core(TM) i7-4770HQ CPU @ 2.20GHz

// Runtime versions
aiace:msgpack parallel$ node -v
v6.8.1
aiace:msgpack parallel$ npm -v
3.10.9

// Module versions
aiace:msgpack parallel$ npm list msgpack
benchmark-msgpack@1.0.0 /Users/parallel/Facile/msgpack
└── msgpack@1.0.2

aiace:msgpack parallel$ npm list fs
benchmark-msgpack@1.0.0 /Users/parallel/Facile/msgpack
└── fs@0.0.1-security

aiace:msgpack parallel$ npm list assert
benchmark-msgpack@1.0.0 /Users/parallel/Facile/msgpack
└── assert@1.4.1
```

# Conclusions{#conclusions}

MessagePack allows to save more than 40% of network bandwidth consumption with little more than one line of code. A smaller payload means that less data are transmitted, and that's very useful in **mobile** and [**Internet of Things (IoT)**](https://en.wikipedia.org/wiki/Internet_of_things) applications, where there's special care in power efficiency; but we should also pay attention to the overall size of each request, to avoid the absurd situation in which the *header* is larger than the *payload* ([overhead](https://en.wikipedia.org/wiki/Overhead_(computing))).

It's important to underline that, while MessagePack is supported by over [50 programming languages](http://msgpack.org/#languages), it doesn't seem to be particularly efficient from a computational perspective, and can be hard to debug due to being non human-readable.
