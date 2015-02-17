---
layout: post
title: "Windows Phone 7 development resources"
date: 2011-09-19 16:13
comments: true
categories: 
 - windows phone
 - programming
 - .net
excerpt_separator: <!--more-->
---

There are a lots of resources and how-to's about Windows Phone 7 development in the Internet. Nevertheless, I though that I'd write down my own list that I'm using. I'm not saying that these are the most complete or best ones but they're the ones I'm mainly using.

<!--more-->

Learning the API and controls
---
 * [Blankenblog: 31 days of Windows Phone 7][blankenblog]: This is an nice collection of short write-ups, perfect for learning new tasks one by one.
 * [MSDN – Windows Phone development][msdnWpDev]: Well d'uh, no surprises here. MSDN docs on WP7 are extensive with plenty of samples etc.
 * [MSDN – Code samples for Windows Phone][msdnWpSamples]: Okay, this is included in the previous one but I wanted to mention it here separately just so you won't miss it. Contains simple samples with good how-to articles on all main features (application bar, local database, background agents, etc.).

MVVM
---
The Model-View-ViewModel pattern is central to developing testable and blendable (applications that work in Blend's designer) applications for WP7. It takes a bit of learning to get started though – especially if you aren't familiar with the concept of Data binding from Silverlight or WPF. To learn about MVVM, check out the MIX10 session “Understanding the Model-View-ViewModel Pattern” by Laurent Bugnion. Laurent is also the author of one of the most popular (if not the most popular, idk) MVVM frameworks, MVVM Light – see “Tools” below for more info. Once you've advanced a little with the MVVM but start to ask questions like “Ok, but what if I want to trigger navigation from the View Model?” or “How do I transmit objects from a VM to another?”, check the follow-up session “Deep Dive MVVM”, held at MIX11. Spending 2 hours on those probably made following the MVVM pattern a lot easier for me.

Tools
---
 * [Development tools][devTools]: Grab the Windows Phone 7 SDK from App Hub. It's got all the things you need for getting started with WP7 development. In case you're wondering, the SDK will not force you to install VS Express if you've already got a paid version of Visual Studio 2010 (SP1).
 * [Expression Blend][blend]: This is included in the Development tools but I'm mentioning it here as I've seen that some people are not aware of it or are avoiding it. Blend is a great tool and a real time-saver when designing UIs and wiring up data templates for WP7. Watching the MIX MVVM sessions by Laurent Bugnion will give you an overview of Blend's capabilities but if you want to learn more, one alternative is to check out the Learning Expression Blend site.
 * [MVVM Light][mvvmLight]: I mentioned this in the MVVM portion. MVVM Light seems to be a quite popular choice of MVVM framework especially in WP7 development. The features aren't anything you couldn't write yourself (probably) but hey, why bother as there's already a free and proven alternative that does the job. Version 4 has a really simple integrated IoC container to reduce the boilerplate code required in the ViewModelLocator, among other things.
 * [Productivity Power Tools extension for Visual Studio][pptExtension]: Hey, it's got “productivity” and “power” next to each other, it must be good! Actually, this extension is a must-have with great features like improved “Add reference” box, pinned tabs, coloured tabs, Quick Find etc. I think the extension will only install on a paid version of VS though, so no free lunch here.

Summary
---

That's about it – the development tools that Microsoft is providing for WP7 (VS 2010 Express & Expression Blend 4) are really good and make designing and developing applications a breeze. Combined with the MVVM pattern and the resulting blendability of the User interfaces, the experience is really enjoyable.

[blankenblog]: http://www.jeffblankenburg.com/2010/09/30/31-days-of-windows-phone-7/
[msdnWpDev]: http://msdn.microsoft.com/en-us/library/ff402535(v=VS.92).aspx
[msdnWpSamples]: http://msdn.microsoft.com/en-us/library/ff431744(v=vs.92).aspx
[devTools]: http://create.msdn.com/en-us/home/getting_started
[blend]: http://www.microsoft.com/expression/windowsphone/
[mvvmLight]: http://mvvmlight.codeplex.com/
[pptExtension]: http://visualstudiogallery.msdn.microsoft.com/d0d33361-18e2-46c0-8ff2-4adea1e34fef