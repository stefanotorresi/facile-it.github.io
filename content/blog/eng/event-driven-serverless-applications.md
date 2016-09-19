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
If we think about computing in the Cloud Era, immediately our mind is focused on virtual machines and containers. Therefore, when we are building a production environment with both approaches, for example, we have to take care of patching the operating system or upgrading the container. At the end of 2014 **Amazon Web Services** (AWS) announced a new service called "**[Lambda](https://aws.amazon.com/lambda/)**", that allows us to focus on business logic and not on infrastructure.

# What is Lambda?{#what-is-lambda}

As reported on their website: « *AWS Lambda is a serverless compute service that runs your code highly-available in the cloud in response to events and it automatically performs all the administration of the compute resources for you* ». In the previous sentence is condensed all the power of Lambda that we can summarize in the following key concepts: **[serverless architecture]({{< ref "#serverless-architecture" >}})**, **[high-availability]({{< ref "#high-availability" >}})**, **[event-driven]({{< ref "#event-driven" >}})** and **[zero administration]({{< ref "#zero-administration" >}})**.

![AWS Lambda key concepts](/images/event-driven-serverless-applications/aws-lambda-key-concepts.png)

## Serverless architecture{#serverless-architecture}

Lambda is completely serverless. The term "serverless" can be considered misleading, obviously Lambda uses physical servers to run your code, but we don’t take care of everything required to do it. We just need to upload our code on AWS console and it handles capacity, scaling, monitoring, logging and security without any server to manage. 
To be strict, functions are executed in containers and in this context kernel-based virtualization is very useful because it allows to build multiple isolated environments in a few time. 
A more detailed explanation of serverless architectures can be found [here](http://www.martinfowler.com/articles/serverless.html).

## High-availability{#high-availability}

AWS Lambda maintains compute capacity across multiple availability zones in each region, at the moment there are 8 regions distributed among Americas, EMEA and Asia Pacific, in this way Lambda helps to protect your code against data center failures.

## Event-driven{#event-driven}

"Event-driven" means that a Lambda function is triggered when an event occurs, so the flow of the application is mainly driven by events. In this kind of architecture all Lambda functions are events consumer, because they are invoked by an event and they are responsible to do something to process it. 
An event comes to life, for example, whenever a new item is created on an Amazon DynamoDB table, a file is deleted on an Amazon S3 bucket, an Amazon API Gateway is called, but we can also use AWS SDK to invoke a function directly on a mobile or web app back-end. 
This is a good way to write application logic without to design and to maintain a centralized workflow. 
More about event-driven programming [here](https://en.wikipedia.org/wiki/Event-driven_programming).

## Zero administration{#zero-administration}

All of the work you need to do to make sure your application works in a scalable, reliable, durable manner is taken care by the service itself.  Behind the scenes the system performs all the administration of the compute resources, including server and operating system maintenance, code and security patch deployment, code monitoring and logging and automatically matches the incoming rate of functions invocation for us, to assure us capacity provisioning and automatic scaling.

# Lambda functions{#lambda-functions}

The code we run on AWS Lambda is called a "**Lambda function**". The name "lambda" derives from the 11th letter of the Greek alphabet. Generally this word is mostly used in programming languages to refer to the syntax whereby you can bind a function in a variable.

## Supported languages{#supported-languages}

Right now Lambda functions support natively code written in **Java**, **Node.js** and **Python**, but we can run C, Go and PHP using a Node.js wrapper. I hope Amazon will add official support for other languages such as PHP, Go, C, Swift and much more. We can also include libraries, even native ones.

## Stateless code{#stateless-code}

When we write a function we must think a **stateless** code, so everything begins and ends in the same request and any persistent state should be stored in a storage service (not necessarily within Amazon world). Keeping function stateless is the keystone to enable the system to be able to instantly launch new instances needed to serve the incoming events rate.

In addition to the code, each Lambda function has many configuration informations, such as name, description, runtime, handler, memory, max execution time and execution role. A detailed explanation is available [here](https://docs.aws.amazon.com/lambda/latest/dg/lambda-introduction-function.html).

## Invocation types{#invocation-types}

We can invoke a Lambda function directly, for example using the Invoke API, or indirectly, for example using the Amazon API Gateway. A function invocation needs to specify the `InvocationType`. There are three invocation types allowed: `RequestResponse`, `Event` and `DryRun`. Each one of them has different purposes:

 * `RequestResponse`: in this case we expect a **synchronous** behaviour. The function receives input parameters as an event and it returns a result;
 * `Event`: in this case we expect an **asynchronous** behaviour. The function receives input parameters as an event, it returns immediately no value, while the function continues its execution;
 * `DryRun`: it is used to verify access to a function without running it.

![Synchronous vs asynchronous behaviour](/images/event-driven-serverless-applications/synchronous-vs-asynchronous-behaviour.png)

# Lambda alternatives{#lambda-alternatives}

Serverless is a new cloud computing trend, accordingly many cloud providers &ndash; in addition to Amazon &ndash; started offering their Function as a Service (FaaS) as for example **Google** with its [Cloud Functions](https://cloud.google.com/functions/), **IBM** with its [OpenWhisk](https://developer.ibm.com/openwhisk/), **Auth0** with its [WebTasks](https://webtask.io/) and **Microsoft** with its [Azure Functions](https://functions.azure.com/).

# Conclusions{#conclusions}

Functions are very useful when we want to build lightweight microservices applications without servers. Their approach could be considered a way to achieve fine-grained microservices, in which there is a relation one-to-one between functions and endpoints instead of one service per one resource, for this reason they are often referred to as "nanoservices".

They could help developers to focus on the code and to only charge for code really running in an infrastructure, that is able to autonomously grow up on demand, without effort in managing it. With functions, we can consider our code as a small and independent building block that can be easily replaced or connected with other blocks using events.

Another consideration is in regard to reuse existing skills or encouraging the adoption of different languages in a small team to develop software that matches better business requirements.
