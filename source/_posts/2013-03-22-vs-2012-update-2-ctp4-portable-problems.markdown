---
layout: post
title: "VS 2012 Update 2 March CTP, WP8 dev tools & portable class libraries - not so fast"
date: 2013-03-22 19:13
comments: true
categories: 
---

New goodies
-----------

The Visual Studio team [recently announced][ctp4] fourth preview of their upcoming "Update 2". As it includes many new features including support for their own [Git plugin][gitPlugin] & [Windows Phone unit testing][wpUnitTest], I wanted to give it a try. 

One other thing that was recently announced is the [Portable HttpClient][portableHttpClient], which basically adds HttpClient support for [Portable Class Libraries][PCL] (PCLs). If you haven't heard about PCLs yet, check them out as they're definitely interesting when doing .NET development for multiple target platforms.

Problems arise
-----------

I was very keen to try these both (we actually had a very good reason to use the Portable HttpClient) so I installed a CTP and started a new project with the HttpClient. Surprisingly, I got compilation errors for this very simple part of code

{% codeblock lang:csharp %}
var client = new HttpClient(); 
HttpResponseMessage response = await client.GetAsync("http://www.google.com"); 
HttpStatusCode httpStatusCode = response.StatusCode;
{% endcodeblock %}

That last line gave me the following errors:

 * The type 'System.Net.HttpStatusCode' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Net, Version=2.0.5.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e, Retargetable=Yes'.
 * Cannot implicitly convert type 'System.Net.HttpStatusCode' to 'System.Net.HttpStatusCode [c:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETPortable\v4.5\Profile\Profile78\System.Net.Primitives.dll]' 

That code is completely valid and any combination of rebuild/clean/restart didn't solve the problem. After going through the uninstall/install/repair route of VS2012 / Update 2 CTP & WP8 dev tools, I had no other explanation to the problems that there has to be something wrong with the update.

So I filed [a bug][bug] about this to Connect. True enough, my woes got answered in a couple of days and (as you can see) after a couple of replies with additional info, Microsoft confirmed that there indeed is a problem when Update 2 CTP 4 is installed after WP8 development tools. Great! Err, or not so great, I don't know. At least now they're aware of it and it wasn't just my that was messed up.

Happily enough, the workaround suggested by Microsoft (repair the Update 2 CTP 4 installation) fixed the issue. 

[ctp4]: http://blogs.msdn.com/b/visualstudioalm/archive/2013/03/04/march-ctp-of-visual-studio-update-2.aspx
[bug]: http://connect.microsoft.com/VisualStudio/feedback/details/781331/vs-2012-update-2-ctp-4-doesnt-work-with-portable-httpclient
[gitPlugin]: http://blogs.msdn.com/b/visualstudioalm/archive/2013/03/06/use-git-0-8-0-0-to-run-scheduled-builds-and-resolve-conflicts.aspx
[wpUnitTest]: http://blogs.msdn.com/b/visualstudioalm/archive/2013/01/31/windows-phone-unit-tests-in-visual-studio-2012-update-2.aspx
[PCL]: http://msdn.microsoft.com/en-us/library/gg597391.aspx
[portableHttpClient]: http://blogs.msdn.com/b/bclteam/archive/2013/02/18/portable-httpclient-for-net-framework-and-windows-phone.aspx