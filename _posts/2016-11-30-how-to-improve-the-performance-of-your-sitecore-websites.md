---
layout: post
title: "How to Improve the Performance of Your Sitecore Websites"
date:   2016-11-30 00:00:00 -0500
categories: sitecore
tags: sitecore performance
author: Mike Skutta
excerpt: A fast loading website is a very important part of any site visitor's experience. Sitecore's Profiler tool can help developers optimize the performance of their website. The Sitecore Profiler tool allows you to debug performance and compare methods while you are developing.
---

A fast loading website is a very important part of any site visitor's experience. Sitecore's Profiler tool can help developers optimize the performance of their website.

The Sitecore Profiler tool allows you to debug performance and compare methods while you are developing.

**Here's what you need to do:**

1. Login as admin.
1. Append this to your querystring: &amp;sc_debug=1&amp;sc_prof=1&amp;sc_trace=1&amp;sc_ri=1
1. Review the performance times of your classes or methods.
1. Investigate classes or methods that are taking a long time and adjust as appropriate.

In addition to seeing the results of time it takes for Sitecore to process and render, you can easily check performances of your own classes or methods.

For example, I want to see whether it&rsquo;s faster to get the item using .GetItem or using .SelectSingleItem

``` csharp
var PathSettings = "/sitecore/content/Sites/Main/WebSettings/Site Settings/Professionals Settings";    

Profiler.StartOperation("GET-ITEM");
Item test1 = Sitecore.Context.Database.GetItem(PathSettings); 
Profiler.EndOperation("GET-ITEM");    

Profiler.StartOperation("GET-ITEM2"); 
Item test2 = Sitecore.Context.Database.SelectSingleItem(PathSettings); 
Profiler.EndOperation("GET-ITEM2");
```

Here's what you'd see on your page as a result:

![improve the performance of sitecore website](/images/how-to-improve-the-performance-of-your-sitecore-websites/improve-performance.png)
