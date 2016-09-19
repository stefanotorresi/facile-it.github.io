---
authors: ["salvatore-cordiano"]
date: "2016-09-19"
draft: true
share: true
categories: [English, Event driven, Serverless, AWS]
title: "Event-driven serverless applications"
type: "post"
languageCode: "en-EN"
twitterImage: '/images/event-driven-serverless-applications/aws-lambda-key-concepts.png'
---
If we think about computing in the Cloud Era, our mind is immediately drawn towards virtual machines and containers. Therefore, for example, when building a production environment with both approaches we think about the need of patching the operating system and/or upgrading the container. At the end of 2014 **Amazon Web Services** (AWS) announced a new service called "**[Lambda](https://aws.amazon.com/lambda/)**", that allows us to focus on business logic and not on infrastructure.

# What is Lambda?{#what-is-lambda}

As reported on their website: « *AWS Lambda is a serverless compute service that runs your code highly-available in the cloud in response to events and it automatically performs all the administration of the compute resources for you* ». In the previous sentence is condensed all the power of Lambda, that we can summarize with the following key concepts: **[serverless architecture]({{< ref "#serverless-architecture" >}})**, **[high-availability]({{< ref "#high-availability" >}})**, **[event-driven]({{< ref "#event-driven" >}})** and **[zero administration]({{< ref "#zero-administration" >}})**.

![AWS Lambda key concepts](/images/event-driven-serverless-applications/aws-lambda-key-concepts.png)

## Serverless architecture{#serverless-architecture}

Lambda is completely "serverless", a term that can be considered misleading: obviously Lambda uses physical servers to run your code, but we, as the service's users, don’t take care of everything required to do it. We just need to upload our code on AWS console and it handles capacity, scaling, monitoring, logging and security without any server to manage. 
Strictly speaking, functions are executed in containers, and kernel-based virtualization is very useful in this context, because it allows to build multiple isolated environments in short time.
A more detailed explanation of serverless architectures can be found [here](http://www.martinfowler.com/articles/serverless.html).

## High-availability{#high-availability}

AWS Lambda maintains compute capacity across multiple availability zones in each region - at the moment there are 8 regions distributed among Americas, EMEA and Asia Pacific - in this way Lambda is able to protect your code against data center failures.

## Event-driven{#event-driven}

"Event-driven" means that a Lambda function is triggered when an event occurs, so the flow of the application is mainly driven by events. In this kind of architecture all Lambda functions are event consumers, because they are invoked by an event and they have the responsibility to process it. 
An event comes to life, for example, whenever:

* a new item is created on an Amazon DynamoDB table;
* a file is deleted on an Amazon S3 bucket;
* an Amazon API Gateway is called;
* et cetera...

but we can also use AWS SDK to invoke a function directly on a mobile or web app back-end. 
This is a good way to write application logic without designing and maintaining a centralized workflow.
More about event-driven programming [here](https://en.wikipedia.org/wiki/Event-driven_programming).

## Zero administration{#zero-administration}

All the work you usually need to do in order to assure that your application works in a scalable, reliable and durable way is taken care by the service itself.  Behind the scenes the system performs all the needed administration for the compute resources, including server and operating system maintenance, code and security patch deployment, code monitoring and logging, and automatically matches the incoming rate of functions invocation for us, to assure capacity provisioning and automatic scaling.

# Lambda functions{#lambda-functions}

The code we run on AWS Lambda is called a "**lambda function**". The name "lambda" derives from the 11th letter of the Greek alphabet. In general a *lambda*, also called *anonymous function*, is a function that's defined inline (sometimes called *closure*) and passed to some other function, method or procedure, to be stored or executed: the *anonymity* is given by the fact that we don't give a name to the function, but we just define it at the moment of need.

## Supported languages{#supported-languages}

Right now Lambda functions natively support code written in **Java**, **Node.js** and **Python**, but we can run C, Go and PHP using a Node.js wrapper. Hopefully Amazon will add official support for other languages such as PHP, Go, C, Swift and many more. We can also include libraries, even native ones.

## Stateless code{#stateless-code}

When we write a function our code must be **stateless**, thus everything begins and ends in the same request, and any persistent state is stored in a storage service (not necessarily within Amazon world). Keeping functions stateless is the keystone to enable the system to instantly launch new instances when needed, to serve the incoming events.

In addition to the code, each Lambda function has many configuration informations, such as name, description, runtime, handler, memory, max execution time and execution role. A detailed explanation is available [here](https://docs.aws.amazon.com/lambda/latest/dg/lambda-introduction-function.html).

## Invocation types{#invocation-types}

We can invoke a Lambda function directly, for example using the Invoke API, or indirectly, for example using the Amazon API Gateway. A function invocation needs to specify the `InvocationType`. There are three invocation types allowed: `RequestResponse`, `Event` and `DryRun`. Each one of them has different purposes:

 * `RequestResponse`: in this case we expect a **synchronous** behavior. The function receives input parameters as an event, and returns a result;
 * `Event`: in this case we expect an **asynchronous** behavior. The function receives input parameters as an event, returns immediately no value, but continues its execution asynchronously;
 * `DryRun`: it's used to verify the access to a function without running it.

![Synchronous vs asynchronous behaviour](/images/event-driven-serverless-applications/synchronous-vs-asynchronous-behaviour.png)

# Lambda alternatives{#lambda-alternatives}

Serverless is a new cloud computing trend, and accordingly many cloud providers – in addition to Amazon - started offering their own *Function as a Service* (FaaS), for example **Google** with its [Cloud Functions](https://cloud.google.com/functions/), **IBM** with its [OpenWhisk](https://developer.ibm.com/openwhisk/), **Auth0** with its [WebTasks](https://webtask.io/) and **Microsoft** with its [Azure Functions](https://functions.azure.com/).

# Conclusions{#conclusions}

Functions are very useful when we want to build lightweight applications based on *microservices* with no server. Their approach could be considered a way to achieve fine-grained microservices, in which there is a relation one-to-one between functions and endpoints instead of one service per one resource: for this reason they are often referred to as *nanoservices*.

They could help developers in focusing on the code, while only charging for code really running in an infrastructure that's able to autonomously grow upon demand, without lots of efforts in managing it. With functions, we can see our code as a series of small and independent building blocks, that can be easily replaced or connected with other blocks using events. Also, Lamba could help small team in reusing existing skills while adopting different languages, in order to develop software that better matches business' requirements.
