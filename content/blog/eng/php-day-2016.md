---
authors: ["jean"]
comments: true
date: "2016-05-17"
draft: true
share: true
categories: [English, PHP, Conferences]
title: "Facile.it devs @ PhpDay 2016"

languageCode: "en-EN"
type: "post"
toc: true
---
As almost a nice tradition, we went to the **[PHP Day conference](http://2016.phpday.it/)** this year as well; it has been held in Verona this time too, on **may 13th to 14th**. 

We partecipated and attended numerous talks and, [like last year]({{< ref "php-day-2015.md" >}}), we would like to propose a light summary; this will not be a full "review", but instead our intent is to highlight what captured our interest most, or what we found more valuable for our everyday work.

In this way, we hope to give a brief glimpse of what we experienced to fellow developers that hadn't the opportunity to attend, or to tempt some of you to join conferences like this one or the PHP community at large, for your personal and professional growth.

The talks are in chronological order, and we try to link the slides when available. Enjoy your reading!

# Technical Talks
## Your API is a UI
 * Christopher Hoult ([@choult](http://twitter.com/choult))
 * Day 1 - 11:00 – 12:00 - track 1
 
This was an interesting and very smooth talk. It stayed pretty general with nice advice about **API development**, but reasoned around a pyramidal "scheme of priorities" that we should follow that derives from the [Maslow's hierarchy of needs](https://en.wikipedia.org/wiki/Maslow%27s_hierarchy_of_needs).

## Dip Your Toes in the Sea of Security
 * James Titcumb ([@asgrim](http://twitter.com/asgrim))
 * Day 1 - 14:30 – 15:30 - track 1 ([slides](http://www.slideshare.net/asgrim1/dip-your-toes-in-the-sea-of-security-phpday-2016))
 
In this talk we saw an interesting overview of **security issues** that a web application can encounter. It ranged from basic SQL injection to timing attacks (that were explained pretty well!) to CSRF protection. 

## How I learned to stop worrying and love Regular Expressions 
 * Jordi Boggiano ([@seldaek](http://twitter.com/seldaek))
 * Day 1 - 15:30 – 16:30 - track 1 ([slides](http://slides.seld.be/?file=2016-05-13+How+I+learned+to+stop+worrying+and+love+Regular+Expressions.html))
 
In this talk we started from basic **regular expressions**, to basic pattern recognition to advanced usage; the speaker took also the time to explain to us how the regex engine works, through thorough examples of how pattern matching advancement and backtracking works. Also, bonus points for the slides! They were pretty neat, and everything was themed to the _Dr. Strangelove_ movie, from images to phrases used in the examples!

## Dockerizing your PHP CI Pipelines
 * Paul Dragoonis ([@dr4goonis](http://twitter.com/dr4goonis))
 * Day 1 - 17:00 – 17:30 - track 2 ([slides](http://dragoonis.com/talks/phpday-may-2016/#/))

This talk was about **using Jenkins with Docker** and having optimized pipelines for fast builds and short commit-to-deploy times. It should have been a 1-hour-long talk, so the speaker had to cut short on a lot of things, but I took a lot of useful advice from it nonetheless. Fortunately, he was kind enough to give the extra bits of his talk in the unconference track the day after.  

## Evolution of Web Application Architecture
 * Kore Nordmann ([@koredn](http://twitter.com/koredn))
 * Day 2 - 15:30 – 16:30 - track 1 ([slides](https://qafoo.com/resources/presentations/phpday_2016_2016/evolution_of_web_application_architecture.html))
 
The talk covered the **evolution of the technological stack** behind a normal web application, from a single server to a multi-server architecture. Each addition to the stack was discussed and weighted, without taking for granted any single step: from adding a master-slave database configuration, to caching session and\or query results. This gave us a pretty clear idea of **when and why** such steps should be taken when enriching a project with new functionalities or overcoming technical challenges.

# Keynotes
## Deploying PHP 7
 * Rasmus Lerdorf ([@rasmus](http://twitter.com/rasmus))
 * Day 1 - 09:45 – 10:45 ([slides](http://talks.php.net/phpday16#/))

Rasmus is a veteran at PHPDay, and he gives opening keynotes here nearly every two yeas. This year's was obviously about PHP 7, and it was more a technical talk than else. It went from **benchmark data** to technical tips on how to smooth the transition to this new major version of PHP.

## How Badoo Saved $1M Switching to PHP7
 * Nikolay Krapivnyy
 * Day 2 - 09:45 – 10:45 ([slides](https://dl.dropboxusercontent.com/u/216377/verona_php_2.pdf))

The [blog post](https://techblog.badoo.com/blog/2016/03/14/how-badoo-saved-one-million-dollars-switching-to-php7/) about this technological switch made the rounds a lot on PHP-related sites some months ago; this talk was a lot technical too, and it showed us how big is Badoo's technological stack, with 3000 servers, of which 1000 running PHP. It gave us a nice overview of the **technical difficulties** that their tech team endured to do this upgrade, and how it gave the the possibility to save more than a half of their hardware resources; also...

<blockquote class="twitter-tweet" data-lang="it"><p lang="en" dir="ltr">Badoo put PHP 7 in production on Friday... <br>/cc <a href="https://twitter.com/agilegigi">@agilegigi</a> <a href="https://twitter.com/hashtag/poiluned%C3%ACcepensiamo?src=hash">#poilunedìcepensiamo</a> <a href="https://twitter.com/hashtag/phpday?src=hash">#phpday</a> <a href="https://t.co/OEj3Emxxee">pic.twitter.com/OEj3Emxxee</a></p>&mdash; Alessandro Lai (@AlessandroLai) <a href="https://twitter.com/AlessandroLai/status/731403882321063936">14 maggio 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
