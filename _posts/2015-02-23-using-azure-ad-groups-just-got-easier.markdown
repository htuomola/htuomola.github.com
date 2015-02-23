---
layout: post
title: "Using Azure AD groups just got easier - and you get roles too"
date: 2015-02-23 21:00:00 +0300
---

In October I blogged about using [Azure AD groups for access control in an ASP.net application][oldPost]. While that post is still valid, using the groups just got easier, as the [Azure team announced][azureAnnouncement] that user's member group information is now returned in the access token itself. Ok, it seems to have been already announced in December but I just learned about it a week ago. 

Even better, they've added support for application roles too. Essentially with roles, the application doesn't have to care about groups and the authorization is done on AAD side by assigning users to roles. It's all covered in that same blog post so head over there for details.

[oldPost]: /blog/2014/10/05/azure-ad-groups-with-webapi-and-owin/
[azureAnnouncement]: http://blogs.technet.com/b/ad/archive/2014/12/18/azure-active-directory-now-with-group-claims-and-application-roles.aspx