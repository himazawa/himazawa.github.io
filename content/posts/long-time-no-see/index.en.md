---
title: "Long Time No See"
date: 2023-02-06T00:01:18+01:00
draft: false
tags: [updates]
featuredImage: header.jpg
categories: [blog-news]
---

Long time no see, uh? Lot of stuff happen since the last post in 2018 on [the old blog](https://bsod.dev).<!--more--> Unfortunately I wasn't able to write much, both for the lack of time and the impossibility to share what I learned.
This post contains an introduction to this new website and how it will work from now on.

First of all, I'm still in Security but moved to Product Security (more on this on a later post but TL;DR: boredom, careeer and growth opportunities, frustration and fear of burnout). 
Fortunately I still have time and resources on my job to conduct vulnerability research so occasionally I will post about interesting vulnerabilities I found.
I would also like to talk about security aspects more related to SDLC, so let's see how it'll go.

## How this blog works
The setup is pretty simple, I'm using [hugo](https://gohugo.io/) to build static pages and publish them on GitHub pages. There is a [Github Action](https://github.com/peaceiris/actions-hugo) running that builds and deploy the pages every time a PR is approved on the `main` branch.
![](blog_CI_CD.png "A graph represeting the logic flow behind the blog deployment system")

The theme used is [LoveIt](https://hugoloveit.com/), it's pretty cool and rich of features but has few bugs that need to be fixed; for example, if you are using the blog in Italian you can see that the localization is pretty much fucked up.
The theme localization it's a pretty cool feature but having to rewrite the same post for each language is painful, so I'm thinking about adding the support to DeepL.

Last but not least, the blog got a new logo:
![](/images/logo.png "appsec.space logo")

it was generated by [Midjourney](https://midjourney.com) and means absolutely nothing!

## The future
What's next, you ask? Well.. a lot of stuff actually. First of all I need to migrate all the posts from the old blog because I would like to retain an archive. Next I can start writing new posts.

{{< admonition note>}}
If you get a redirect from the old blog to the new one the process has been completed.
{{< /admonition >}}

Despite being called `appsec.space` I would like to talk about different topics, spacing from Application Security to Malware Developmemnt (for learning purposes [of course](https://www.youtube.com/watch?v=HmZm8vNHBSU)), to productivity setup and generic rants, basically turning this blog into my stream of counciousness.

The next post will probably talk about the [Security Theater](https://en.wikipedia.org/wiki/Security_theater).

See you then!