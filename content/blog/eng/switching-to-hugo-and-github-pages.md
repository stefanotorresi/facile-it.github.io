---
authors: ["jean"]
comments: true
date: "2016-04-26"
draft: false
share: true
categories: [English, Blog, Hugo, Static site, GitHub]
title: "Blog restyling: switching to Hugo and GitHub Pages"

languageCode: "en-US"
type: "post"
---
As you may have noticed, we went through a bit of a restyling in the last few days, and our blog changed a lot in terms of appearance and structure. 
We are very happy with those changes, but the reason behind this transition is not just a simple template change.

## What we were searching for
Before, we were using an hosted CMS to run the blog, [Ghost](https://ghost.org/). We were happy with the results, it wasn't our concern to administrate the server or bother about other hosting stuff.

But, as many other developers, we were **hitching for something** more aligned with our inclinations: we wanted a tool that was closer to our daily jobs and our skills. We were also having **problems during the revision phase** of the articles, since multiple persons couldn't review and edit an article at the same time, fearing the risk of overwriting each other's work. 

We then came down with a list of requirements for our blog; we wanted:

 * **easy review of articles**: parallels reviews are a must
 * **an easy workflow**: no major overhead, since the blog contributions are voluntary amongs us; we want to dedicate all the time to writing the articles, not wrestling with technical difficulties
 * **no security flaws**: this point disqualified Wordpress and other famous CMS solutions; we wouldn't like to spend time staying ahead of possible security risks, or have to worry over the hosting and installation problems that a widespread used CMS has
 * **easy image hosting** and embedding
 * easy management and modifications of the **template**
 * easy integrations of previously used [Disqus comments](https://disqus.com/) and Google Analytics
 * preferably **markdown content format**
 * support for **multilanguage articles**

## Git and the GitHub pages
The first solution that came to us was obviously **Git**, the everyday tool for a team of developers working on the same piece of code (or text, in this case). We obviously then thought of the [GitHub pages](https://pages.github.com/) as an hosting solution: that is ideal, since we already have a [Github organization](https://github.com/facile-it) in place, so an organization homepage/site would perfectly fit the bill for our blog.

## Hugo
Once we set our eyes to that hosting solution, we started to search for a solution for building our blog. The proposed solution was [Hugo](http://gohugo.io/): it's a static site generator, developed in Go, which fitted almost all of our requisites; also:

 * it generates the static HMTL site live, with a **[live reload](https://gohugo.io/extras/livereload/) functionality**: while you edit your markdown article, the site refresh itself on each save of the file(s), so you can see a live preview of what you're writing
 * there are many templates ready to be used, and we [chose one](http://themes.gohugo.io/future-imperfect/) to be modified to our needs
 * the templates are built with the Go Templating engine, and with some easy manual edits we obtained all the requested features: multi-author support, link between translated articles, recent articles and top categories in the sidebar...

## Deploying
The last step needed to smooth out our workflow was to find a **deploy strategy**. Github pages need to have the site content published onto the master branch, and we obviously wanted to separate the source code of the site from that. 

We needed an easy way to reproduce this steps, ideally with a script commited to the repo, ready for everyone to use:

 * **generate** the static site with Hugo
 * **avoid mixing (and committing)** the static copy with the source files
 * commit the files to the master branch
 
Hugo normally generates the static copy in the `./public` directory; to achieve this we used [a script that I found on GitHub itself](https://github.com/X1011/git-directory-deploy) that solved this exact problem. We started an empty `source` branch, **detached** from master:
 
```
git checkout -b source --orphan
```
 
We started our Hugo site inside it, and we put the `./public` in the `.gitIgnore`, so each static generation would not be committed. We then prepared the deploy script that, after generating the static site, uses a Git subtree approach to commit its content to the master branch, with a commit message of `publish: <last commit message from source>`.

## Et voilà, here we are!

So, we hope that you will enjoy our new blog at least as much as we enjoyed building it ! Happy reading!
