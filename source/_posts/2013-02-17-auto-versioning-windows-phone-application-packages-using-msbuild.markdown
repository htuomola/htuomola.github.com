---
layout: post
title: "Auto-versioning Windows Phone application packages using MSBuild"
date: 2013-02-17 14:22
comments: true
categories: 
- windows phone
- programming
---
	
**TL;DR;** To auto-version your Windows Phone XAP files, set AssemblyVersion attribute value to e.g. "1.2.*" and use the MSBuild AfterBuild snippet in step 2 TODO LINK.

One problem with testing and gathering feedback from software that is not deployed centrally (e.g. websites) but distributed to testers and other people is to know which version of the application is in question. There is always someone that hasn't updated to the latest version and of course we want to avoid chasing after issues that have already been solved.

I know that there are a lot od options for collecting diagnostic and analytical data from WP applications (automatic uploads, email, 3rd party services) but that is not the topic of this post. Here I'm merely presenting a way to have each different build of your WP application o be *uniquelly identifiable* to help with diagnosing issues.

1st step: automatic assembly numbering
---
The first step of course is to have the application builds numbered. In C# projects this can be easily done by changing the [AssemblyVersion attribute][asverAttr] value of the application to form "n.m.*", where n and m are normal numbers, e.g. 1.2.*. This instructs MSBuild (the build engine behind .NET projects) to provide the assembly with auto-incrementing build number. 

From first look the build number seems completely random - something like TODO - but what actually happens is that the third number is incremented each day and the fourth number is unique, auto-incrementing *for that day*. 

Note: the first two numbers are not auto-incremented with this approach and I think they should anyway be manually managed when new versions of the application are released to end-users.

Note2: when specifying AssemblyVersion like above you should comment out AssemblyFileVersion completely if you wish that bothfiles have the same version number. The latter has some limitations with its support for auto-increment versioning TODO VERIFY. If, however, the AssemblyFileVersion is not set at all it seems to get the same value as AssemblyVersion.

Step 2: adding version info to the Windows Phone application package
---
If we were distributing only assemblies (DLLs) what we've done would already be enough because the version of an assembly can be easily verified theough file properties:

TODO IMAGE

As you probably know if you're reading this, the output of a WIndows Phone application project is a XAP file and unfortunately it doesn't contain similar version info as assembly files. The workaround I've used is to add build version to the XAP file's name.

{% gist 4434914 %}

NOTE: This script intentionally also leaves the original XAP file to the build output folder. I first tried to just rename it but that leads to the debugging not being able to start! Apparently Visual Studio needs that file for deploying it to the emulator and updating the XapFilename property value to the new filename doesn't help! So in the end you'll have two copies of the XAP file, one can be diatributed to testers etc. and the other is used for debugging.

[asverAttr]: http://msdn.microsoft.com/en-us/library/system.reflection.assemblyversionattribute(v=vs.110).aspx