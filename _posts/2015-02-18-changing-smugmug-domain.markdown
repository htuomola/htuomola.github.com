---
layout: post
title: "Changing Smugmug domain, Azure websites & IIS to the rescue"
date: 2015-02-18 22:40:00 +0300
---

I recently decided to start to migrate my web presence from domain mnd.fi to henrituomola.fi. Actually I've had the new domain for a while now but I just haven't moved old stuff over.

So I started changing my Smugmug site custom domain from [http://photos.mnd.fi](http://photos.mnd.fi) to [http://photos.henrituomola.fi](http://photos.henrituomola.fi). Smugmug deals with custom domains as all other sites, you configure the domain to use and then point (in this case) a CNAME record to there. I thought that I'd be clever and modified photos.mnd.fi CNAME record to point to photos.henrituomola.fi &mdash; which itself was a CNAME record pointing to domains.smugmug.com.

The problem
===
So after doing these changes I tested the new domain and it worked great. Then I tested the old domain and just got a Smugmug "landing page" instead of my page. After some wondering I realized that due to the way CNAME records are handled, when photos.mnd.fi was requested Smugmug only saw that domain and not photos.henrituomola.fi. After my config change on their side, Smugmug didn't have a clue what photos.mnd.fi was supposed to relate to.

The solution
===
I realized that I have to setup a HTTP redirect from the old domain to the new one.

Redirecting to new domain with IIS 
---

To do this, I created a new empty ASP.net website and setup IIS Url Rewrite rules to redirect all traffic to the new domain. Redirect can be implemented with the following addition to web.config file:

{% highlight xml %}
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Redirects to new site" stopProcessing="true">
          <match url=".*" />
          <action type="Redirect" url="http://photos.henrituomola.fi/{R:0}" />
        </rule>
      </rules>
    </rewrite>
    <modules runAllManagedModulesForAllRequests="true" />
  </system.webServer>
{% endhighlight %}

[The documentation on this][iisUrlRewrite] is somewhat spread out and hard-to-find but if you've got IIS installed, you can enter these using the GUI. I didn't so I had to google and experiment for a moment.

Gone - permanently
----
One thing you want to consider with redirects is the their kind. The [HTTP spec][http] lists the following relevant status codes in the redirection category: Moved Permanently, Found, See Other, Temporary Redirect. In this kind of case you want to return "Moved Permanently" (301) in order to convey to browsers, search engines etc. that they should start using the new location now.

While some sources (for IIS 7) mention that type "Redirect" default status code would be "Temporary", I tried it out and it seems to return 301 Moved Permanently. So I was all set with this configuration. There's currently no need to care about the requested address as I want to redirect all requests.

Live with Github and Azure website
---
For hosting the source codes I published it to Github. Then I created a new Azure Website and configured it to continuously deploy from my Github repo. Within minutes I had the site deployed. Then I only had to register photos.mnd.fi domain to my Azure Website and I was all set. You can see the full solution in [the github repo][githubRepo]. Azure Websites are great and play nicely with Github, see [here][azureWebsites] for more info and [here][azureWebsitesCI] on how to configure continuous deployment.

[http]:  http://en.wikipedia.org/wiki/List_of_HTTP_status_codes#3xx_Redirection
[githubRepo]: https://github.com/htuomola/PhotosRedirect
[iisUrlRewrite]: http://www.iis.net/learn/extensions/url-rewrite-module/url-rewrite-module-configuration-reference#Redirect_action
[azureWebsites]: http://azure.microsoft.com/fi-fi/services/websites/
[azureWebsitesCI]: http://azure.microsoft.com/fi-fi/documentation/articles/web-sites-publish-source-control/