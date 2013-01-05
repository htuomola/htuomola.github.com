---
layout: post
title: "Site history & Going static with Octopress &amp; GitHub pages"
date: 2013-01-05 17:14
comments: true
categories: 
 - blog
 - web devevelopment
---

History
-------

My personal website (this right here) has had several incarnations. Way back, in the beginning of 00's I think the first implementations were using PHP with some PHP photo gallery apps and a combination of static HTML. 

### An era of WordPress

After a while I - like most of people doing PHP, I presume - realised that PHP *blows* as a language. During those times WordPress was very popular and I decided to for once not write my own website but use WordPress as the publishing platform. I still think it's a good blogging platform. I reckon I started with v2.0 or v2.1 and the first versions had some issues and annoyances but at least most of them were sorted out in the newer versions. 3.0 felt pretty mature already.

<!--more-->

### Orchard CMS

In 2011 I was planning an update of the site layout and theme. I wanted to make drastic changes to the site structure and that seemed to be a bit challenging with the existing WordPress setup. Eventhough WordPress claimed to be a Content Management System (CMS) it didn't really feel like that. Also, WordPress internals were still using PHP and MySQL with me being a .NET/C# developer  bugged me out.

I'm not sure if I considered writing my own site at this point but while browsing the internet for alternatives I saw [Orchard CMS] [1]. Orchard CMS is a free, open-source CMS system that's been built using ASP.NET MVC, NHibernate & others. It's being actively maintained by people ([@bleroy][bleroy], [@sebastienros][sebastienros], others) who work on the ASP.NET team at Microsoft.

ASP.NET MVC 3 was brand new at that time and I hadn't looked into it so looking into Orchard offered a way to do two things at once. I did realize that Orchard has quite a steep learning curve especially if you want to understand the inner workings - the architecture has been made very extensible and as a result of this it's quite difficult to get in to. Quoting from their [mission statement](http://orchardproject.net/mission):

> Orchard is built on a modern architecture that puts extensibility up-front, as its number one concern. All components in Orchard can be replaced or extended. Content is built from easily composable building blocks. Modules extend the system in a very decoupled fashion, where a commenting module for example can as easily apply to pages, blog posts, photos or products. A rich UI composition system completes the picture and ensures that you can get the exact presentation that you need for your content.

It did take me a couple of weekends and evenings to get the site setup like I wanted but I was quite happy with the outcome.

### Going custom, again

As I said, I was pretty happy with Orchard CMS until fall 2012 when the guys at my hosting company ran some Windows security updates and whatnot to the environment I was running Orchard CMS on. As a result of those my site broke down totally, it was just returning internal server errors with some mysterious loading problems. One part of the problem might've been that I didn't have the latest version of Orchard - as of now the upgrade process involves quite a lot of manual steps and I just couldn't have been bothered with it.

I think it was a Saturday evening after that when I got the idea that why not roll my own site (again). So I created a new website using ASP.NET MVC 4 (using Razor of course) & Entity Framework 5 code first. I already knew them somewhat so it didn't take long to get the basic structure built up. Then I plugged in the Twitter Bootstrap nuget package and the site started to look quite nice. After mainly adjusting the CSS styles I decided that it was good enough and published it to replace my old site. Note that I did leave out the blog 'for now' as I wanted to get the site back up again.

Source codes for this version are now available on [GitHub](https://github.com/htuomola/mndfi-mvc).

A static blog, what the?!
--

### Setup

Right, to get to the main point of this article, an idea came up with when I had to add the blog back to my site/blog. Markdown syntax and static site generators like [Jekyll](https://github.com/mojombo/jekyll) have become popular recently. Jekyll is actually powering GitHub pages, which makes hosting "kind of dynamic" sites on GitHub a breeze.

A friend of mine, Aki Saarinen, had already blogged about [setting up a website with Jekyll, Compass and Twitter Bootstrap][2]. After reading that I wasn't quite convinced that I wanted to leave my nice .NET world ;-) and go through all the work he'd done. Then I found out about [Octopress](http://octopress.org/), which is a framework built on top of Jekyll to provide blogging capabilities and basic HTML templates out-of-the-box. That sounded too good to not try it out.

True enough, I had a bare-bones blogging site running on my computer in half an hour - of which ~25 minutes was spent installing ruby to Windows 8. Finally when I found [yari][yari], everything went smoothly.

### Deployment

Deploying Octopress especially to [GitHub Pages](http://pages.github.com/) was like a walk in the park and only involves a few steps, as [described here](http://octopress.org/docs/deploying/github/).

[1]: http://www.orchardproject.net/
[2]: http://akisaarinen.fi/blog/2012/04/09/running-this-site-with-jekyll-compass-and-twitter-bootstrap/
[bleroy]: https://twitter.com/bleroy
[sebastienros]: https://twitter.com/sebastienros
[yari]: https://github.com/scottmuc/yari
    