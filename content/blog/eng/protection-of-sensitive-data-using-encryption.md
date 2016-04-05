---
authors: ["sandro"]
comments: true
date: '2015-10-05'
draft: true
image: "/images/cover.jpg"
menu: ""
share: true
categories: [English, Encryption, RSA, AES, Rjindael]
title: "Protection of sensitive data using encryption"

languageCode: "en-En"
type: "post"

---
Data protection is one of the major and recurrent problems in recent years: whether it is private information of users, or the company for which we work, the problem is always the same.

**How to protect such data from any attackers who would - and could (!!) - be able to gain possession?**

Before arriving at the solution of this problem, however, is right to split the series in at least two branches. In the wwww data can be "obtained" in two different moments: 1) as they pass over the network in packets, or 2) after their storing (eg. database or file) in one or more servers.

This article will explore the second case, and will be proposed a solution: how to protect the data that we have accumulated sparingly "in house" ?.

## Putting data in the safe

In case of stored data, for example, within a database; it is certainly more difficult to achieve our treasure, because they are "safe" in a protected environment, which we trust more than the volatility of the net.
However it is better not to feel too sure: the more our data will be valuable, as any interested parties will be encouraged to seek every way to obtain them.
Any flaw, both hardware and software, however limited in time could expose our wealth to various attacks, allowing anyone (or almost) to get them.

A possible and effective protection technique in these cases is **making useless or unreadable data** for those who have obtained them in an illegitimate manner, through their storage in encrypted form.

What we propose below is a possible solution to the problem of how to store and encrypt this data, in order to obtain an adequate level of safety and maintain good elasticity, so as to allow different implementations and to cover as many possible use cases.

The same solution provides that the first part of the exposed problematic (data protection during transit) is resolved, and will be based on the combined use of two different types of encryption algorithms.

## Cryptographic Algorithms Symmetric and asymmetric

With cryptographic algorithms ** refer to all processes and procedures aimed at obtaining a given "clouded" so as not to be understandable / intelligible ** by persons not authorized to read it.

### Symmetric Algorithms 

This type of algorithm is based on the use of encryption keys said symmetrical that allow  **to encrypt and decrypt data using the same cryptographic key**.
An example is [AES] (https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (Advanced Encryption Standard) algorithm, evolution of [Rijndael] (https://en.wikipedia.org/wiki/Rijndael_key_schedule) the whose characteristics can be summarized in a good level of safety and excellent speed, whether it is implemented in hardware or software.

![AES](/images/protezione-di-dati-sensibili-usando-la-crittografia/aes.png)

### Asymmetric Algorithms 

The asymmetric algorithms are distinguished from the previous to the use of two distinct ** cryptographic keys, said public and private, to perform the encryption and decryption operations.
The name derives from the method by which the two keys are to be used. The public key is freely exchanged, **The private key is known only to those who have to be able to read** data or messages exchanged.
The feature that characterizes these keys is the impossibility of obtaining, for example, the private key being in possession of the public key. Unless a particular algorithm exploits, the only method to obtain this key is the brute force **on that data.
In this case the example for excellence is [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)), a de facto standard in the security on data transmission: it is in fact widely used to encrypt the communications that take place between the client and server.

![RSA](/images/protezione-di-dati-sensibili-usando-la-crittografia/rsa.png)

## The solution combining RSA and AES
Now that we have rehearsed the technological basis, we can think about how to implement a solution.

Imagine we have a platform that allows various users to connect with their credentials and perform various operations, including storing their own private documents (photos, documents, passwords, contracts, etc.) that need a more high level of security, and maybe we can share this information with each other in the near future.

From these simple conditions, and preparing all the future "sharable" document, a possible solution is the following:

* We will use AES to encrypt individual documents;
* For the sharing, we will use a table (exchange) of our database; inside the owner of the document will be tracked and users can access it;
* Each user will have his key ring containing the RSA keys that serve as "lock" for the individual document key;
* You will need to also obfuscate the private key of each user, so as to allow access only to the owner;

### Managing Users
At the creation of a new user, we must **generate a new RSA key pair** , which will go tied to the user.
Before storing these keys in the database (along with the user's credentials on another table), the private key will need to be obfuscated. It is good practice, in these cases, to let the user to choose a **masterkey****, to be used in turn to encrypt the private key with a symmetric algorithm.
Masterkey chosen will be very important as it will be required each time we have to decipher our data (unfortunately, the user experience is sacrificed in favor of a much higher level of security).
**N.B.** It is strongly recommended not to store the masterkey (in any form) on the server: the attacker could easily access it and groped to force unlocking consequently all the other keys.

### Encrypt Document
**Each time** a user will upload a file or insert a text to be protected, you will need to generate a new **unique** symmetric key. We will use the key to encrypt the document using AES and store it in our database.

### Associate the file to the user
As a last step we have to associate the encrypted data to the user who is storing them. It's time to use the "exchange table". We will insert a record containing references to the user and the file, together with the unique key tied to it. But the key will be encrypted before using the user's public key.

We got a similar system to the one shown below:

![System](/images/protezione-di-dati-sensibili-usando-la-crittografia/system.png)

### Share!
Each time a document (or who has obtained the right to read it) owner user wants to share it with a second user we will simply do the following:

* Require user masterkey;
* Decrypt the private key of the same user using the key you just obtained;
* Decipher the key document related to (recovered from the exchange table);
* Make a copy of the exchange record, replacing the user with the target and re-encrypting the key exchange with the public of the recipient;

### And now?

Now that we have completed (at least theoretically) our data protection system **we are definitely safe?** **Absolutely not** Leaving aside the smooth technical operation carried out so far, we must remember that we have entrusted much of **our system security to the user** .
The weakness lies in the fact masterkey is left in the hands of our users: if they were to share it, or lose it, it would be impossible to guarantee the security or the recovery of stored data.
Also a key - which is user's choice - too simple would be easy to circumvent through brute force or other techniques.

Finally, I want to urge you to comply with all [general rules concerning the composition of the password] (https://en.wikipedia.org/wiki/Password#Factors_in_the_security_of_a_password_system), avoid users to choose a too simple one.
