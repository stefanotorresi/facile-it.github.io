---
authors: ["nicola"]
comments: true
date: "2016-06-20"
draft: true
share: true
categories: [English, Agile, Scrum, Kanban]
title: "An Agile workflow"

languageCode: "en-EN"
type: "post"
---
Talking about **agile methodologies** is very difficult. First of all, because it is not possible to define a generic method that is universally valid. I am used to listen that "with SCRUM you’ll sure to get good results". I think this is incorrect.

Lots of people assume that the values and principles of the [Agile Manifesto](http://www.agilemanifesto.org/) are just a collection of "ready to use recipes" to achieve good results in a software development process. What is "*agile*"  with this approach? It's like cooking with a kitchen robot. For this reason, I am sure that the perfect recipe is something to discover "testing" your favorite ingredients combined together (I’ll continue with cooking jokes... You’ve been warned!).

Some likes simple recipes, like a *grill*. For example, some groups like a very present manager, a boss that tells to every member of the team what he wants them to do, step by step. All is guided but efficient. 
Others love sophisticated organization, like a complex recipe of a *starred restaurant*. Very complicated systems to track communications and time organization (remote teams often needs procedures and tools, I admire this kind of complexity).
In the end, we have groups that love *fast food*. Fast food is the same everywhere and all people love it! Like SCRUM. 
So, someone can assume that SCRUM is a good recipe because many people like standard recipes. But, are we sure that the typical fast food sandwich is the best way to eat?

## Learn by own mistakes
Our group have crossed some months trying to apply SCRUM. Just trouble! We were looking where we were failing without find the right way. We get a lots of stress and conflict and our retrospectives seems like an Alcoholics Anonymous meeting rather than feedbacks time. Now we know some things that we mistake:

 * Our inside client needs to change requirements faster than a single one-week iteration. These generate too many frustration and we were feeling that every iteration was failing.
 * We had just lose some important and senior developers. Our domain is very complicated e we were in trouble with estimation. Every week we were reducing the quality of our codebase with technical debt and the quality of our work life.
 * We didn’t understand how important is the SCRUM Master. Someone that protect the values and ceremonies of this practice. We moved between iteration without celebrate success or try to change something.

## Change recipe
I think that be an Agile Developer is, first of all, don’t be trap in some mindsets. So, during an iteration, I stopped the team and I told them that these was not my think about Agile. We started to look for a different recipe, something right for our needs. First we looked characteristics of our group:

 * Priority change every day.
 * We want quality and high technological component.
 * Our team is very good but many of us don’t know domain well.
 * We don’t know our velocity and we think that this parameter will change every week with our understanding of domain.
 * We can’t stop develop because there are a lots of bugs and client needs our release.

The team is composed by five developers and a product specialist (a team member that collect all need and bugs from client and help us to keep process useful).
We open a Google Drive Spreadsheet to track user stories from the product specialist and for every story we put a label to describe the domain (frontend, invoice, etc.) and a summary estimate (from little story to epic). This is our backlog.

At the begin of the month we use to have a backlog review with the manager to find priority and we remove stories in conflict with business manager. An important ingredient is continuous adaptation so we choose to use Kanban to view our workflow. We use Trello to see our stories (tasks) moving in the board. The first column is "next steps": we have not iteration, we use to put here stories when the column is empty, in generally from backlog or directly by the clients (continuous prioritization!). 

Every story contains a label from the backlog, so every developer can understand what kind of task is looking to catch (often a new member of team can make just some kind of simple tasks). When a developer is available, he "adopt" a story and move a card in "doing" column and put his avatar on the card. Doing a story for us is: get information from client, check mockups, develop, create a demo environment and release. When the task is finish and released we use to move the card in the column "to be checked in production". After some days without problems, we could consider the task really "done" and we move it in the appropriate column.

For every story we try to adopt good practice like unit and functional test. We check our activity with a continuous integration tools. New domains stories are the most critical because me must made many important choices related on project architecture. I think that pair programming it is nota technique to twice release speed rather than a way to halve "stupidity" and mistakes of developer! Anyone can do something wrong but promote a mistake together is rare, probably someone will try to have challenge for important choice.
If there are some problem with a story we use to help team member in trouble with two o three developer to move the story to done (this is Kanban practice).

Every two weeks, at less one member of the team protects the other developers from bug, problem and various stuff needs by clients and management. This kind of intrusion could damage our iteration plans. This developer protects the team from intrusions. We call him "wallman"! When wallman is available from intrusion there is always some technical debt to be fired or something to be refactored. Our team, following Lean way, release a lot of quick experimental features. Sometimes are manipulation of preexistent codes. When the market gives us positive feedbacks, wallman helps us to clean code. So, the team can use time to develop new business features when wallman protect them from bugs.

This recipe is the combination of many ingredients. We like that! I’m not sure that is good in every situation and every people but we think that is good and respect our values (and values by Agile Manifesto).

## The perfect recipe
I think that like in cooking, when every good recipe has a little part sweet and a part sour, every combination of practice must follow fundamentals principles. Put them in your world! All need challenge, all need retrospectives and in the end, right combination will forget to you fast food and will discover a great homemade sandwich! 
