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
When we talk about data interchange in web applications, **[JSON](http://www.json.org/)** is the de-facto standard particularly if we think to develop a RESTful web services. JSON won his antagonist [XML](https://www.w3.org/XML/) (SOAP) without battle, but it has not deterred the creation of alternatives like [Google’s **Protocol Buffers**](https://developers.google.com/protocol-buffers/), [Apache **Avro**](https://avro.apache.org/) or **[MessagePack](http://msgpack.org/)**. In order to be complete we can also mention a [gzip](http://www.gzip.org/) compressed version of JSON (sometimes also called "*JSONC*") and [BSON](http://bsonspec.org/) that is a bin­ary-en­coded seri­al­iz­a­tion of JSON-like doc­u­ments, both derived directly from JSON. In this post we will be discussing MessagePack in greater depth.

![MessagePack](/images/message-pack-an-alternative-to-json/msgpack.png)

# What is MessagePack?{#what-is-messagepack}

« *MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller* ». To start using MessagePack we need to convert our application objects into MessagePack formats, this process is called *serialization*; vice versa the reverse process is called *deserializazion*. The following example helps us to better understand what we're talking. Consider this simple JSON:

```json
{
	"id": 4,                    // integer
	"isActive": true,           // boolean
	"fullname": "Homer Simpson" // string
}
```

JSON requires 56 bytes to represent a very simple user object, instead using MessagePack it becomes only 38 bytes (compression ratio 1.47, it yields a space saving of 32%), see below the output of serializazion process of the above JSON:

```
83 a2 69 64 04 a8 69 73 41 63 74 69 76 65 c3 a8 66 75 6c 6c 6e 61 6d 65 ad 48 6f 6d 65 72 20 53 69 6d 70 73 6f 6e
```

We can comprehend how MessagePack serialization works reading the [official specification](https://github.com/msgpack/msgpack/blob/master/spec.md). Inter alia, we can split the previous hexadecimal representation to emphasize and explain data types as follows:

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

Now is very simple to figure out the meaning of the sentence: « *Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves* » reported in the headline of MessagePack website.

The main features of MessagePack are:

- It's designed for network communication and to be transparently converted from and to JSON;
- It supports in-place updating, so it's possible to modify part of stored object without reserializing whole of the object;
- It has a flexible [Remote Procedure Call (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call) and streaming API implementation;
- It supports [static-typing](https://en.wikipedia.org/wiki/Type_system) and [type-checking](https://en.wikipedia.org/wiki/Type_system#Static_type_checking).

# Supported data types{#supported-data-types}

Data structures listed by the specification are very similar to JSON data types and they are:

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

Until the moment our reasoning is focused on **space efficiency**, but a good theoretical computer scientists would have criticized us, because of we omitted to mention the **time complexity**. In fact the process of data *compression* and *decompression* it is not negligible. So we can analyse and compare, for example, the time necessary to parse a JSON document and to unpack a MessagePack document. This is not a scientific method, but it could help us not to overlook the timespent.

We wrote two **[Node.js](https://nodejs.org/en/)** scripts to execute 1 Milion of JSON parsing and 1 Milion of MessagePack unpacking of a document containing the same data in the two rispectively different formats. We use as [sample document](https://github.com/salvatorecordiano/facile-it-message-pack-benchmark/blob/master/document.json) bigger than the first JSON of this post. 

Simplifying our code, we can write something like this:

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

To easily profiling our scripts we can run them as below:

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

Numbers are self explanatory: the binary MessagePack is *smaller* than minified JSON, but MessagePack deserialization is *slower* than JSON parsing process, without a doubt.

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

MessagePack allows to save more than 40% of network bandwidth consumption with little more than one line of code. A smaller payload means less data are transmited, this is very useful in **mobile** and [**Internet of Things (IoT)**](https://en.wikipedia.org/wiki/Internet_of_things) applications, where attention to the use of the battery is very important, but we should pay attention on the size of each request to avoid the absurd situation in which the *header* is larger than data in *payload* ([overhead](https://en.wikipedia.org/wiki/Overhead_(computing))).

It is important to underline that MessagePack is supported by over [50 programming languages](http://msgpack.org/#languages) although from the computational point of view not particularly efficient and it is not especially simple to debug because it isn't human-readable.
