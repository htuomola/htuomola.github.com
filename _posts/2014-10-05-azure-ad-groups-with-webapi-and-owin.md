---
layout: post
title: "Azure AD group-based authorization with WebAPI and OWIN"
date: 2014-10-05 11:05:41 +0300
comments: true
categories: azure programming .net
featured: true
---

I blogged about using Azure AD (AAD) groups as roles in an ASP.net MVC application [a while ago][aadRolesPost]. While speaking at Namesdays in Espoo, Finland last week, I presented briefly how the same approach can be applied in a WebAPI project. If you want to use AAD groups for adding role-based authorization to your APIs built with ASP.net WebAPI the story is similar but not identical.

Because in my case both solutions are based on the OWIN middleware, we can use the same approach of hooking up to OWIN "user authenticated" step for claims transformation - or maybe I should say enrichment.

Instead of using `CookieAuthenticationOptions` and `CookieAuthenticationProvider`, we can hook up to `OAuthBearerAuthenticationProvider.OnValidateIdentity` callback:

{% highlight csharp %}
public void ConfigureAuth(IAppBuilder app)
{
	app.UseWindowsAzureActiveDirectoryBearerAuthentication(
		new WindowsAzureActiveDirectoryBearerAuthenticationOptions
		{
			Audience = ConfigurationManager.AppSettings["ida:Audience"],
			Tenant = ConfigurationManager.AppSettings["ida:Tenant"],
			Provider = new OAuthBearerAuthenticationProvider()
			{
				OnValidateIdentity = AuthenticationOnValidateIdentity
			}
	});
}
{% endhighlight %}

The membership verification and claims process is identical to the in in [the previous post][aadRolesPost] but here it is again

{% highlight csharp %}
private async Task AuthenticationOnValidateIdentity(OAuthValidateIdentityContext context)
{
    try
    {
        const string graphApiResourceId = "https://graph.windows.net";
        const string oidClaimType = "http://schemas.microsoft.com/identity/claims/objectidentifier";
        const string financeGroupId = "<enter group's object id from Azure AD portal>";
        const string claimIssuer = "WebApiApp";

        string tenantName = ConfigurationManager.AppSettings["ida:Tenant"];
        string authority = string.Format("https://login.windows.net/{0}", tenantName);
        string clientId = ConfigurationManager.AppSettings["ida:ClientID"];
        string clientSecret = ConfigurationManager.AppSettings["ida:Password"];

        var authContext = new AuthenticationContext(authority);
        var credential = new ClientCredential(clientId, clientSecret);

        // Get token to access AAD graph API
        var token = authContext.AcquireToken(graphApiResourceId, credential);
        
        var graphConnection = new GraphConnection(token.AccessToken);

        // check group membership from AAD
        var currentUserObjectId = context.Ticket.Identity.Claims.First(c => c.Type == oidClaimType).Value;
        var isMemberOf = graphConnection.IsMemberOf(financeGroupId, currentUserObjectId);
        
        // add Role claim to support asp.net Authorize attribute roles' validation
        if(isMemberOf)
            context.Ticket.Identity.AddClaim(new Claim(ClaimTypes.Role, 
                RestrictedGroupName, ClaimValueTypes.String, claimIssuer));
    }
    catch (Exception e)
    {
        // TODO: error logging
    }
}
{% endhighlight %}

As previously, this code needs both Microsoft.Azure.ActiveDirectory.GraphClient and Microsoft.IdentityModel.Clients.ActiveDirectory nuget packages to work.

NOTE! The above code doesn't cache group membership and it will check it on every request - even if the currently requested API method wouldn't require user to be a member of (any) group. Also Azure AD access tokens for checking the membership is also cached in-memory as that's the default in ADAL. Please see [this post][adalTokenCache] on more information about ADAL token caching.

[aadRolesPost]: http://mnd.fi/blog/2014/08/18/using-azure-ad-groups-as-roles-with-owin/
[adalTokenCache]: http://www.cloudidentity.com/blog/2014/07/09/the-new-token-cache-in-adal-v2/