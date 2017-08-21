---
layout: post
title: "Sorting Sitecore Items by Popularity using Google Analytics"
date:   2015-05-20 00:00:00 -0500
categories: sitecore
tags: sitecore google analytics sorting
author: Mike Skutta
excerpt: Not too long ago (prior to Sitecore 7.5) we had a requirement to add the ability to sort Sitecore items by Most Viewed and Most Shared. We got to thinking about what was the best way to implement something like this. In the past we may have created a custom database to track this information. Upon each page view or share, we could have written a row to the database. This data would then be available for use in sorting. This approach has costs involved with needing to maintain an extra database and an API. Could we take a simpler approach? We were already using Google Analytics to track page views and social interactions. Google provides APIs for its products, so we should be able to retrieve the data we need. We decided to go down the route of using Google Analytics to provide us with the data required for sorting by Mode Viewed and Most Shared.
---

* content
{:toc}

## Overview

Not too long ago (prior to Sitecore 7.5) we had a requirement to add the ability to sort Sitecore items by Most Viewed and Most Shared. We got to thinking about what was the best way to implement something like this. In the past we may have created a custom database to track this information. Upon each page view or share, we could have written a row to the database. This data would then be available for use in sorting. This approach has costs involved with needing to maintain an extra database and an API. Could we take a simpler approach? We were already using Google Analytics to track page views and social interactions. Google provides APIs for its products, so we should be able to retrieve the data we need. We decided to go down the route of using Google Analytics to provide us with the data required for sorting by Mode Viewed and Most Shared.

## Google Analytics

At the most basic level, Google Analytics provides the ability to track page views. This is achieved by including a tracking code on every page you want to track. The tracking code calls out to the Google Analytics servers registering information about the page view. Beyond page views, Google Analytics also had the ability to trackSocial Interactions. [Social interactions](https://developers.google.com/analytics/devguides/platform/social-interactions) are user interactions with social buttons and widgets. Clicking the Facebook “Like” button would register a Social Interaction. Generally, event handlers need to be manually wired to the buttons or widgets to register the social interaction. Social Activities take Social Interactions to the next level. [Social Activities](https://developers.google.com/analytics/devguides/platform/social-activities-overview) track Social Interactions on other sites that reference your site. This allows a blog post reference on another site to be reported back to the blog owner.

For this implementation, we will only be addressing Page Views and Social Interactions. I just wanted to mention Social Activities to show what else can be achieved.

## Tracking Code

As I mentioned before, Google Analytics was already being used on the site we wanted to add sorting by Page Views and Social Interactions to. Going through the Analytics Core Reporting API, we could query for whatever data we were interested in. The [Dimensions & Metrics Reference](https://developers.google.com/analytics/devguides/reporting/core/dimsmets) is a good reference for what data is available through the API. Using the reference, we see that we want **ga:pageviews** and **ga:socialInteractions** metrics to get the number of page views and social interactions. The dimensions within the **Page Tracking** section gives us information about the page that was requested. We could use the **ga:pagePath** to figure out what the related Item was. This would require figuring out what item lives at the specific URL or path. This felt like there would be a lot of overhead to find the Item from the path. If we could include the Item ID as part of the page title, then we could quickly parse out the item id without needing to look it up. From this, we can use the **ga:pageTitle** dimension.

How do we get the item ID into the page title? Google Analytics provides Tracking Code for page views. This code needs to live on each page that is tracked. The code typically looks like this:

``` html
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
  ga('create', 'UA-XXXXXXXX-Y', 'auto');
  ga('send', 'pageview');
</script>
```

We can extend the above code adding the Item ID. To do so, we can use the set command.

``` html
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
  ga('create', 'UA-41806526-2', 'auto');
  ga('set', 'title', document.title + ' | {4e8d0e01-15a1-47da-ab59-57c704caf1ed}');
  ga('send', 'pageview');
</script>
```

We are essentially keeping the existing title and appending the Item ID with a pipe separator. This is still human readable if viewing in the Google Analytics reports and it is also parsable. The GUID should be replaced with the current Sitecore Item ID. The approach for getting the Item ID varies based on MVC, Web Forms, and/or the implementation. Now that we have set the **title**, we should have the Item ID available to us for all Page Views and Social Interactions.

I am not going to cover how to add the Social Interaction script for Google Analytics because it is dependent on the implementation. See [here](https://developers.google.com/analytics/devguides/collection/analyticsjs/social-interactions) for more details on how to implement.

## Getting the Results

Now that the data is being collected correctly with the associated Item ID, we need to be able to query for the results. Google provides a convenient Client Library for interacting with Google Analytics data. You can add this client library through NuGet. The library is named [Google.Apis.Analytics.v3](http://www.nuget.org/packages/Google.Apis.Analytics.v3/)

The first thing we need to do is provide configuration settings for access to the data from with your Google Analytics account. These settings include the Profile Id, Service Account Email, and the Key File Path. You can learn more about where to obtain the values for these settings from the [Readme.md](https://github.com/onenorth/social-sort) on the GitHub repository. The last setting is the Window in Days. These are the number of days to look back when retrieving analytics data. In our particular case we just want to look at current analytics data. We generally just want to use the past 30 days of data.

Here is the code to get the settings

``` csharp
namespace OneNorth.SocialSort.Configuration
{
    public class Settings : ISettings
    {
        public string GoogleAnalyticsProfileId
        {
            get { return Sitecore.Configuration.Settings.GetSetting("OneNorth.SocialSort.GoogleAnalytics.ProfileId", ""); }
        }

        public string GoogleAnalyticsServiceAccountEmail
        {
            get { return Sitecore.Configuration.Settings.GetSetting("OneNorth.SocialSort.GoogleAnalytics.ServiceAccountEmail", ""); }
        }

        public string GoogleAnalyticsKeyFilePath
        {
            get { return Sitecore.Configuration.Settings.GetSetting("OneNorth.SocialSort.GoogleAnalytics.KeyFilePath", ""); }
        }

        public int WindowInDays
        {
            get { return Sitecore.Configuration.Settings.GetIntSetting("OneNorth.SocialSort.WindowInDays", 30); }
        }
    }
}
```

Now that we have the settings, we can use the Google Analytics API to retrieve the data. First, we need to obtain a reference to the Analytics Service. To do this we need to P12 file path and Service Account Email from the settings.

``` csharp
private AnalyticsService GetService()
{
    // Google Analytics API Service Account Authentication 
    var keyFilePath = _settings.GoogleAnalyticsKeyFilePath;    // found in developer console under APIs & auth / Credentials
    var serviceAccountEmail = _settings.GoogleAnalyticsServiceAccountEmail;  // found in developer console under APIs & auth / Credentials
    var certificate = new X509Certificate2(keyFilePath, "notasecret", X509KeyStorageFlags.Exportable); // notasecret is the standard password for the key file.
    var credential = new ServiceAccountCredential(new ServiceAccountCredential.Initializer(serviceAccountEmail)
    {
        Scopes = new[] { AnalyticsService.Scope.AnalyticsReadonly }
    }.FromCertificate(certificate));

    // Google Analytics Service
    var service = new AnalyticsService(new BaseClientService.Initializer
    {
        HttpClientInitializer = credential,
        ApplicationName = "OneNorth.SocialSort", // This can be whatever you want
    });

    return service;
}
```

Once we have the reference to the service, we can build the request object. The request includes the profile id from the settings. It also includes the data range we want to retrieve data for. Lastly it includes the Metrics and Dimensions we want to retrieve; **ga:pageviews**, **ga:socialInteractions**, and **ga:pageTitle**

``` csharp
private DataResource.GaResource.GetRequest GetRequest(AnalyticsService service)
{
    // format the profile id
    var profileId = _settings.GoogleAnalyticsProfileId;
    if (!profileId.Contains("ga:"))
        profileId = string.Format("ga:{0}", profileId);

    var window = _settings.WindowInDays;
    var startDate = DateTime.Now.AddDays(-window);
    var endDate = DateTime.Now;

    var request = service.Data.Ga.Get(profileId, startDate.ToString("yyyy-MM-dd"), endDate.ToString("yyyy-MM-dd"), "ga:pageviews,ga:socialInteractions");
    request.Dimensions = "ga:pageTitle";

    return request;
}
```

We can then execute the request. Google may return pages worth of data. because of this, we need to build in a way to retrieve the results page by page. We want to return the results as a dictionary, where the key is the Item ID. The value holds the number of page views and social interactions.

``` csharp
private Dictionary<Guid, Metrics> Execute(DataResource.GaResource.GetRequest request)
{

    // Retrieve data, performing paging if necessary.
    var metrics = new Dictionary<Guid, Metrics>();
    GaData response = null;
    do
    {
        var startIndex = 1;
        if (response != null && !string.IsNullOrEmpty(response.NextLink))
        {
            var uri = new Uri(response.NextLink);
            var paramerters = uri.Query.Split('&');
            var s = paramerters.First(i => i.Contains("start-index")).Split('=')[1];
            startIndex = int.Parse(s);
        }

        request.StartIndex = startIndex;
        response = request.Execute();
        ProcessData(response, metrics);

    } while (!string.IsNullOrEmpty(response.NextLink));

    return metrics;
}
```

Now that we have the results, we need to parse them. Remember, the page title not only contains the actual page title, but also the Item ID. The results are returned as column headers and rows. we need to figure out what column contains which Dimension or Metric.

``` csharp
private void ProcessData(GaData response, Dictionary<Guid, Metrics> metrics)
{
    var pageTitleIndex = 0;
    var pageViewsIndex = 0;
    var socialInteractionsIndex = 0;

    // Find associated columns
    for (var index = 0; index < response.ColumnHeaders.Count; index++)
    {
        var header = response.ColumnHeaders[index];

        if (string.Equals(header.Name, "ga:pageTitle", StringComparison.OrdinalIgnoreCase))
            pageTitleIndex = index;
        else if (string.Equals(header.Name, "ga:pageviews", StringComparison.OrdinalIgnoreCase))
            pageViewsIndex = index;
        else if (string.Equals(header.Name, "ga:socialInteractions", StringComparison.OrdinalIgnoreCase))
            socialInteractionsIndex = index;
    }

    foreach (var row in response.Rows)
    {
        // Try to get the item id from the page title
        var pageTitle = row[pageTitleIndex];
        var parts = pageTitle.Split('|').Select(x => x.Trim()).ToArray();

        Guid itemId;
        if (!Guid.TryParse(parts.Last(), out itemId))
            continue;

        // Get page views
        int pageViews;
        if (!int.TryParse(row[pageViewsIndex], out pageViews))
            pageViews = 0;

        // Get Social Interactions
        int socialInteractions;
        if (!int.TryParse(row[socialInteractionsIndex], out socialInteractions))
            socialInteractions = 0;

        if (!metrics.ContainsKey(itemId))
            metrics.Add(itemId, new Metrics { PageViews = pageViews, SocialInteractions = socialInteractions });
        else
        {
            var entry = metrics[itemId];
            entry.PageViews += pageViews;
            entry.SocialInteractions += socialInteractions;
        }
    }
}
```

## Caching the Results

The above code retrieves the analytics data for an entire site covering a 30 day period. It would be beneficial to not retrieve this data each time we need to use it. Analytics data is not that time sensitive. Caching Analytics data for a short period should not make that big of an impact to the overall results. Because of this, we should be able cache the analytics data. If we refreshed the cache once an hour, users should see reasonable results. We can use Sitecore’s Task Scheduler to run a task once an hour to update the Analytics data cache.

First we need to create a class that manages the cache.

``` csharp
internal class Cache
{
    private static Dictionary<Guid, Metrics> _metrics;

    static Cache()
    {
        _metrics = new Dictionary<Guid, Metrics>();
    }

    internal static void CacheMetrics(Dictionary<Guid, Metrics> metrics)
    {
        // This is atomic and we dont need locking.
        _metrics = metrics;
    }

    public static int GetPageViews(Guid id)
    {
        Metrics metrics;
        return (_metrics.TryGetValue(id, out metrics)) ? metrics.PageViews : 0;
    }

    public static int GetSocialInteractions(Guid id)
    {
        Metrics metrics;
        return (_metrics.TryGetValue(id, out metrics)) ? metrics.SocialInteractions : 0;
    }
}
```

Now we just need to create the Task and have the task call all of the helper methods we went over above.

``` csharp
public class Task
{
    public void Run()
    {
        var manager = new Manager();
        manager.UpdateCaches();
    }
}

public class Manager : IManager
{
    public void UpdateCaches()
    {
        var service = GetService();
        var request = GetRequest(service);
        var metrics = Execute(request);
        Cache.CacheMetrics(metrics);
    }
}
```

Lastly we need to build out our configuration file. This file not only includes the settings for the Google Analytics API, but also the scheduled task.

``` xml
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <settings>
      <setting name="OneNorth.SocialSort.GoogleAnalytics.ProfileId" value="" />
      <setting name="OneNorth.SocialSort.GoogleAnalytics.ServiceAccountEmail" value="" />
      <setting name="OneNorth.SocialSort.GoogleAnalytics.KeyFilePath" value="" />
      <setting name="OneNorth.SocialSort.WindowInDays" value="30" />
    </settings>
  </sitecore>
  <scheduling>
    <agent type="OneNorth.SocialSort.GoogleAnalytics.Task, OneNorth.SocialSort" method="Run" interval="01:00:00" />
  </scheduling>
</configuration>
```

## Usage

To make things easier to use, we will create a helper service with several overloads.

``` csharp
public class Service : IService
{
    public int GetPageViews(Item item)
    {
        return GetPageViews(item.ID);
    }

    public int GetPageViews(ID id)
    {
        return GetPageViews(id.Guid);
    }

    public int GetPageViews(Guid id)
    {
        return Cache.GetPageViews(id);
    }

    public int GetSocialInteractions(Item item)
    {
        return GetSocialInteractions(item.ID);
    }

    public int GetSocialInteractions(ID id)
    {
        return GetSocialInteractions(id.Guid);
    }

    public int GetSocialInteractions(Guid id)
    {
        return Cache.GetSocialInteractions(id);
    }
}
```

Using the above helper methods, we can easily use the Analytics data to sort. To obtain the number of page views and social interactions for an Item, you can do the following:

``` csharp
var service = new OneNorth.SocialSort.GoogleAnalytics.Service();

// Obtain the number of page views for an item
var pageViews = service.GetPageViews(<Item>);

// Obtain the number of social interactions for an item
var socialInteractions = service.GetSocialInteractions(<Item>);
```

As you can see, it is very simple to get the number of page views and social interactions for an item. These methods pull from cache and therefore perform well. We can expand this example to sorting a list of Sitecore Items based on the number of views.

``` csharp
var service = new OneNorth.SocialSort.GoogleAnalytics.Service();

List<Item> items = <Get list of items from somewhere>;
var sortedItems = items.OrderBy(service.GetPageViews);
```

## Conclusion

As you can see, it is quite simple to get Google Analytics Page View and Social Interaction data for use in Sitecore. All of the code for these examples as well as additional information is located here: [https://github.com/onenorth/social-sort](https://github.com/onenorth/social-sort)