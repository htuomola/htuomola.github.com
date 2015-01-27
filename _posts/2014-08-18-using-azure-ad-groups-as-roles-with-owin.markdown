---
layout: post
title: "Using Azure AD groups as roles with OWIN"
date: 2014-08-18 18:14:41 +0300
comments: true
categories: programming azure .net
featured: true 
---

I've been working with Azure Active Directory and claims-based authentication on several occasions lately. I like the model and where it's all going but the fast development pace of Azure AD combined with the ever-changing ASP.net authentication introduces some new problems from time to time. 

One such problem on what I didn't find good documentation or a blog post on was how to do group-based authorization in Azure AD. Azure AD has had the concept of groups for a while now and especially in an enterprise scenario they're a common way to restrict access to resources. 

In my case I was using the ASP.net MVC 5 & [OWIN](http://owin.org/ "OWIN — Open Web Interface for .NET") middleware and the it's authentication setup is different, albeit more simple than the previous configuration-based approach. So after a while of searching it became apparent that I can still use the ASP.net MVC `[Authorize(Role="Admin")]` attribute for authorization, I just needed to inject the roles as claims myself. Most of the idea has already been documented at [this blog post](http://www.azurefromthetrenches.com/?p=911 "Azure AD, Groups, Roles and the Authorize Attribute").

That post, however, doesn't use OWIN but the old approach and the *claims transformation* phase is different in OWIN as there's now ClaimsAuthorizationManager (I think). Or at least I didn't see configuration for one anywhere so changing it might be problematic. Then I ended up reading [Tero's blog post](http://teelahti.fi/the-promise-of-owin-starts-to-materialize/ "The promise of OWIN starts to materialize") about claims transformation in OWIN and the picture started to be complete.

So in the end I've added a callback on `CookieAuthenticationProvider.OnResponseSignIn` and read user's roles from Azure AD at that point. I'm only checking a membership of a single predefined role so I can use the [isMemberOf Graph API method](http://msdn.microsoft.com/en-us/library/azure/dn151601.aspx). After adding NuGet packages [ADAL](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/ "Active Directory Authentication Library") and [Graph API client library](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient/ "Microsoft Azure Active Directory Graph Client Library ") and a bit of coding, the final solution looks like this:

{% highlight csharp %}
public class StartUp 
{
	internal const string AdminRoleName = "Admin";
	private const string ClaimIssuer = "SomeInternalApp1";
	private const string AdminGroupId = "<enter object id of admin group in Azure AD>";
	private const string ClientId = "<enter Azure AD application client ID>";
	private const string ClientKey = "<enter Azure AD application client key>";

	public void ConfigureAuth(IAppBuilder app)
	{
		app.SetDefaultSignInAsAuthenticationType(WsFederationAuthenticationDefaults.AuthenticationType);
		app.UseCookieAuthentication(
			 new CookieAuthenticationOptions
			 {
				 AuthenticationType = WsFederationAuthenticationDefaults.AuthenticationType,
				 Provider = new CookieAuthenticationProvider
				 {
					 OnResponseSignIn = ctx =>
					 {
						 ctx.Identity = TransformClaims(ctx.Identity);
					 }
				 }
			 });
		[ .. more configuration as usual .. ] 
	}
			 
	private ClaimsIdentity TransformClaims(ClaimsIdentity identity)
	{
		// this check isn't probably necessary if only called from OnResponseSignIn but doesn't hurt either
	    if (identity.IsAuthenticated)
	    {
	        var userObjectId = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
	        if (IsMemberOfAdminGroup(userObjectId))
	            identity.AddClaim(new Claim(System.Security.Claims.ClaimTypes.Role, 
			AdminRoleName, ClaimValueTypes.String, ClaimIssuer));
	    }
	    return identity;
	}		
		
	private bool IsMemberOfAdminGroup(string userObjectId)
	{
	    try
	    {
	        // Get the access token
	        var authContext = new AuthenticationContext(_authority);
	        var credential = new ClientCredential(ClientId, ClientKey);
	        AuthenticationResult result = authContext.AcquireToken("https://graph.windows.net", credential);

	        var clientRequestId = Guid.NewGuid();
	        var graphSettings = new GraphSettings {ApiVersion = "2013-04-05"};
	        var graphConnection = new GraphConnection(result.AccessToken, clientRequestId, graphSettings);
	        // check group membership
	        bool isMemberOf = graphConnection.IsMemberOf(AdminGroupId, userObjectId);
	        return isMemberOf;
	    }
	    catch (Exception e)
	    {
	        _logger.ErrorException("An exception occurred while checking admin group membership", e);
	        return false;
	    }
	}
}
{% endhighlight %}

Now in the controller it's possible to restrict access using the same approach as with normal ASP.net roles or on-premise AD groups:

{% highlight csharp %}
[Authorize(Role=StartUp.AdminRoleName)]
public class TopSecretController : Controller 
{ 
	[...] 
}
{% endhighlight %}

This is especially nice from the controller point-of-view as no code changes were needed to read the roles from Azure AD.

Disclaimer: This code is for demonstrating the idea only. It has several points of improvements: standard ADAL in-memory token cache is used, role membership is only checked when logging in, etc. YMMV. 