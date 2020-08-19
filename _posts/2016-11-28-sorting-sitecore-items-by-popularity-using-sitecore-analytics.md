---
layout: post
title: "Sorting Sitecore Items by Popularity Using Sitecore Analytics"
date:   2016-11-28 00:00:00 -0500
categories: sitecore
tags: sitecore hosting
author: Mike Skutta
excerpt: A while back, I created a blog post that explained how to Sort Sitecore Items by Popularity using Google Analytics. In this post, I will explain how to achieve the same goal using xDB. As a quick recap, clients may have a requirement to sort Sitecore items by Most Viewed. In many instances, Sitecore Analytics and xDB are already being used to track analytics information.  We should be able to retrieve the data we need from xDB as well as the reporting database. There may be alternate ways to retrieve the data from xDB and the reporting database.  I am going to take the route that includes adding a Page Event and a custom Fact Table, demonstrating how they function.
---

* content
{:toc}

## Overview

A while back, I created a blog post that explained how to [Sort Sitecore Items by Popularity using Google Analytics](/2015/05/20/sorting-sitecore-items-by-popularity-using-google-analytics/). In this post, I will explain how to achieve the same goal using xDB.

As a quick recap, clients may have a requirement to sort Sitecore items by Most Viewed. In many instances, Sitecore Analytics and xDB are already being used to track analytics information. We should be able to retrieve the data we need from xDB as well as the reporting database.

There may be alternate ways to retrieve the data from xDB and the reporting database. I am going to take the route that includes adding a Page Event and a custom Fact Table, demonstrating how they function.

## Setting the Scene

The client has a requirement to list the most viewed items based on a given template type in the past month. In English: the client wants to display the most viewed Articles in the past month.

## Approach

To achieve this requirement, we need to track the number of views each item has had. To get results by template, we need to track the template IDs and then filter by them. "In the past month" is a sliding window. The easiest way to handle this sliding window is to keep track of the total views per day.

## Page Event

We want to track the template ID associated with the Item viewed. The template ID is not included in the Interaction data in xDB. Looking up items by template ID will be much faster if the template ID is already included in the Analytics Data.

> Note: You can view exactly what is being tracked per interaction by viewing the data in Mongo DB. To do this, you can use a tool such as Robomongo. Using the tool, open the Sitecore "analytics" Mongo Database. View the documents in the "Interactions" collection. Expanding each document will show you what is tracked per interaction. Within the document, expand the "Pages" node. You should see all the pages visited during this interaction. If you drill into a page, you can see what is tracked per page.

Using a page event is one way to add additional information to an item when it is viewed. We can use a page event to track the template ID of the item. To add a page event, we need to register it within Sitecore. In the Content tree (master database), navigate to System / Settings / Analytics / Page Events. From there, create a new Page Event and give it a name of "View".

![Sorting Sitecore Items by Popularity using Sitecore Analytics](/images/sorting-sitecore-items-by-popularity-using-sitecore-analytics/sortingsitecore1.png)

Once the page event is added in Sitecore, we need to register the event when an item is viewed.&nbsp; First, create a pipeline processor to that registers the page event.

*ViewPageEventProcessor.cs*
``` csharp
using System;
using Sitecore.Analytics;
using Sitecore.Diagnostics;
using Sitecore.Pipelines;
using Website.Logic.Common.Extensions;
using Sitecore.Data;
using Newtonsoft.Json;

namespace Website.Logic.Analytics.Views
{
    public class ViewPageEventProcessor
    {
        public void Process(PipelineArgs args)
        {
            Assert.ArgumentNotNull(args, "args");
            Assert.IsNotNull(Tracker.Current, "Tracker.Current is not initialized");
            Assert.IsNotNull(Tracker.Current.Session, "Tracker.Current.Session is not initialized");
            Assert.IsNotNull(Tracker.Current.Session.Interaction, "Tracker.Current.Session.Interaction is not initialized");
            if (Tracker.Current.Session.Interaction == null)
                return;
            var currentPage = Tracker.Current.CurrentPage;
            if (currentPage == null || currentPage.IsCancelled)
                return;

            var item = Sitecore.Context.Item;
            if (item == null)
                return;

            var pageEventData = currentPage.Register(new Sitecore.Analytics.Data.PageEventData("View")
            {
                ItemId = item.ID.ToGuid(),
                Text = string.Format("Template ID: {0}", item.TemplateID),
                DataKey = item.TemplateID.ToString(),
                Data = item.TemplateID.ToString()
            });
        }
    }
}
```

Notice in the above example that the Data is the template ID.

We now need to add the pipeline processor to the &lt;endAnalytics&gt; pipeline.

*ViewPageEventProcessor.config*
``` xml
<?xml version="1.0"?>
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <endAnalytics>
        <processor type="Website.Logic.Analytics.Views.ViewPageEventProcessor, Website" patch:before="processor[@type='Sitecore.Analytics.Pipelines.EndAnalytics.SaveDuration, Sitecore.Analytics']"/>
      </endAnalytics>
    </pipelines>
  </sitecore>
</configuration>
```

Now that the Page Event is registered in Sitecore, and a custom pipeline has been configured, we should see the View Page Event appear in Mongo DB within the Page Interactions.

> Note: you may not see your interactions until your session ends and the results are written to xDB. You can programmatically update the analytics database without ending the visitors session by following the example here.

## Fact Table

In order to more easily retrieve the data we are logging via page events, we should aggregate this data into a Fact table in the Sitecore Reporting database. The data will be aggregated by day and item. From this we will be able to tell how many times an item was viewed per day. We will also keep track of the Template ID for quick lookups.

First, create a table in the reporting database that contains the ItemId, Date, TemplateId and Count. The reporting database is usually the one with "analytics" in its name.

*Fact_Views.sql*
``` sql
ALTER TABLE [dbo].[Fact_Views] DROP CONSTRAINT [FK_Fact_Views_Items]
GO

/****** Object:  Table [dbo].[Fact_Views]    Script Date: 8/22/2016 8:44:14 AM ******/
DROP TABLE [dbo].[Fact_Views]
GO

/****** Object:  Table [dbo].[Fact_Views]    Script Date: 8/22/2016 8:44:14 AM ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[Fact_Views](
	[ItemId] [uniqueidentifier] NOT NULL,	
	[Date] datetime NOT NULL,
	[TemplateId] [uniqueidentifier] NOT NULL,
	[Count] [bigint] NOT NULL
 CONSTRAINT [PK_Fact_Views] PRIMARY KEY CLUSTERED 
(
	[ItemId] ASC,
	[Date] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

GO

ALTER TABLE [dbo].[Fact_Views]  WITH NOCHECK ADD  CONSTRAINT [FK_Fact_Views_Items] FOREIGN KEY([ItemId])
REFERENCES [dbo].[Items] ([ItemId])
GO

ALTER TABLE [dbo].[Fact_Views] CHECK CONSTRAINT [FK_Fact_Views_Items]
GO

CREATE NONCLUSTERED INDEX [IX_Fact_Views_ByTemplateDate] ON [dbo].[Fact_Views]
(
	[TemplateId] ASC,
	[Date] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)
GO

```

Now that we have our table, we can create the aggregator that populates the table with data. Sitecore needs a Fact&lt;&gt; class defined that supports the new fact table. The fact class and related classes are defined as follows:

*ViewFact.cs*
``` csharp
using Sitecore.Analytics.Aggregation.Data.Model;

namespace Website.Logic.Analytics.Views
{
    public class ViewFact : Fact<ViewKey, ViewValue>
    {
        public ViewFact() : base(ViewValue.Reduce)
        {
            
        }

        public override string TableName
        {
            get { return "Views"; }
        }
    }
}
```

*ViewKey.cs*
``` csharp
using System;
using Sitecore.Analytics.Aggregation.Data.Model;

namespace Website.Logic.Analytics.Views
{
    public class ViewKey : DictionaryKey
    {
        public Guid ItemId { get; set; }
        public DateTime Date { get; set; }
    }
}
```

*ViewValue.cs*
``` csharp
using Sitecore.Analytics.Aggregation.Data.Model;
using System;

namespace Website.Logic.Analytics.Views
{
    public class ViewValue : DictionaryValue
    {
        internal static ViewValue Reduce(ViewValue left, ViewValue right)
        {
            var viewValue = new ViewValue();

            viewValue.Count = left.Count + right.Count;
            viewValue.TemplateId = left.TemplateId
            
            return viewValue;
        }

        public long Count { get; set; }
        public Guid TemplateId { get; set; }
    }
}
```

These classes can then be used within an AggregationProcessor that handles aggregating the data from xDB into the reporting database.

*ViewAggregationProcessor.cs*
``` csharp
using System;
using System.Linq;
using Sitecore.Analytics.Aggregation.Pipeline;
using Sitecore.Diagnostics;

namespace Website.Logic.Analytics.Views
{
    public class ViewAggregationProcessor : AggregationProcessor
    {
        protected override void OnProcess(AggregationPipelineArgs args)
        {
            Assert.ArgumentNotNull(args, "args");

            if (args.Context.Visit.Pages == null)
                return;

            foreach (var page in args.Context.Visit.Pages)
            {
                if (page.PageEvents != null)
                {
                    var fact = args.GetFact<ViewFact>();
                    foreach (var pageEvent in page.PageEvents.Where(p => p.Name == "View"))
                    {
                        var viewKey = new ViewKey {
                            Date = pageEvent.DateTime.Date,
                            ItemId = pageEvent.ItemId
                        };

                        var viewValue = new ViewValue
                        {
                            Count = 1,
                            TemplateId = new Guid(pageEvent.Data)
                        };

                        fact.Emit(viewKey, viewValue);
                    }
                }
            }
        }
    }
}
```

This processor needs to be registered via configuration.

*ViewAggregationProcessor.config*
``` xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <group groupName="analytics.aggregation">
        <pipelines>
          <interactions>
            <processor type="Website.Logic.Analytics.Views.ViewAggregationProcessor, Website" />
          </interactions>
        </pipelines>
      </group>
    </pipelines>
  </sitecore>
</configuration>
```

Now as analytics data is collected, the Fact_Views table should be populated with aggregated data.&nbsp; You can test this out by navigating your site and watch the Fact_Views table populate.

## Gathering the List of Top Items

Now that the analytics data exists in a form we can work with, it is time to query the data.&nbsp; We can create a helper method to retrieve the data.

*MostViewedHelper.cs*
``` csharp
public static List<Item> GetMostViewed(int count, int daysToInclude, List<Guid> templateIDs, List<Guid> mostViewedExcludeGuids)
{
    var sqlServerDataApi = new SqlServerDataApi(Sitecore.Configuration.Settings.GetConnectionString("reporting"));

    // Base Query
    var query = @"SELECT TOP " + count.ToString() + @"
                    A.ItemId,
                    A.DailyCount,
                    A.LastViewDate
                FROM
                (
                SELECT	ItemId, 
                        SUM([Count]) AS DailyCount,
                    Max(Date) AS LastViewDate
                FROM    Fact_Views 
                WHERE   ViewDate >= DATEADD(DAY, -" + daysToInclude.ToString() + @", GETDATE()) ";

    // Restrict To Search Templates
    if (templateIDs != null && templateIDs.Any())
    {
        query += @"AND     TemplateId IN (" + string.Join(",", templateIDs.Select(x => string.Format("'{0}'", x.ToString()))) + @") ";
    }

    // Exclude Specific ItemIDs
    if (mostViewedExcludeGuids != null && mostViewedExcludeGuids.Any())
    {
        query += @"AND     ItemId NOT IN (" + string.Join(",", mostViewedExcludeGuids.Select(x => string.Format("'{0}'", x.ToString()))) + @") ";
    }

    // Group And Sort
    query += @"GROUP BY ItemId
                ) AS A
                ORDER BY A.DailyCount DESC,
                    A.LastViewDate DESC";

    var result = new List<Item>();

    var reader = sqlServerDataApi.CreateReader(query);
    while (reader.Read())
    {
        var itemID = reader.InnerReader.GetGuid(0);
        var scItem = Sitecore.Context.Database.GetItem(new ID(itemID));
        if (scItem != null)
        {
            result.Add(scItem);
        }
    }
    return result;

}
```

These results should be cached to improve performance.&nbsp; I will leave it up to the reader to cache the results based on requirements.

## Conclusion

As you can see, it is pretty straightforward to use the data from Sitecore Analytics to find the Items that are most viewed.