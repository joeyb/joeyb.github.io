---
layout: post
title: "Quick Code Samples: Building a Simple Config Service"
tags: [quick-code-samples, csharp]
---

I really hate hard-coding values into applications. Inevitably requirements change and it's generally much easier to just change a value in the web.config than to re-build and deploy new code. Because of that, I like to make heavy use of config files.

In order to avoid the pitfalls that I mentioned in my article on [tight coupling][1], I've built an `IConfigService` interface that exposes the needed functionality and an `AppSettingsConfigService` implementation of that interface. In my case, all I care about is fetching values from the config (as opposed to storing values back in the config) so this is the interface that I use:


{% gist 1683091 IConfigService.cs %}


The `AppSettingsConfigService` implementation takes advantage of `Convert.ChangeType()` to generically convert the string value that we get back from `AppSettings` into the type that was requested.


{% gist 1683141 AppSettingsConfigService.cs %}


Now you can just setup your IoC container to resolve the `IConfigService` interface to the `AppSettingsConfigService` implementation and begin pulling back config values.

After seeing this code, I know a lot of developers will give the following response:

> Why not just use ConfigurationManager.AppSettings directly?
> What benefit is there to adding another layer of abstraction?

I agree that in your initial build of the app there may be no benefit to this service, but it really shines when requirements change. For example, let's say that your client now wants to move all of these config values to a database. If you had used `AppSettings` directly in your code then it's going to be a nightmare to make the requested change. But if you programmed to the interface, then all you would need to do is build a new `IConfigService` implementation that fetches the value out of the database, then update your IoC container to resolve `IConfigService` to the new implementation and the rest of your code just works. An extra few minutes up front saved hours of pain down the road, sounds like a great trade-off to me.

[1]: {% post_url 2012-01-25-tight-coupling-a-brownfield-project-nightmare %}
