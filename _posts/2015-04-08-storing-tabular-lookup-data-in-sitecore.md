---
layout: post
title: "Storing Tabular Lookup Data in Sitecore"
date:   2015-04-08 00:00:00 -0500
categories: sitecore
tags: sitecore tabular data
author: Mike Skutta
image:
    url: /images/storing-tabular-lookup-data-in-sitecore/StoringTabularLookupDataInSitecore_Spreadsheet.png
    height: 241
    width: 862
excerpt: In some cases, there may be a need to store tabular data in Sitecore. For example, you may need to store pricing information for a product based on quantity.
---

* content
{:toc}

## Overview

In some cases, there may be a need to store tabular data in Sitecore. For example, you may need to store pricing information for a product based on quantity.

| Product | per lb | 3+ lb | 5+ lb | 8+ lb |
|---------|---|---|---|---|
| Apples | 1.13 | 0.81 | 0.80 | 0.80 |
| Bananas | 0.49 | 0.40 | 0.38 | 0.37 |
| Oranges | 1.00 | 0.83 | 0.73 | 0.72 |

In this example, we could have a price per pound based on the total number of pounds. You can see in the table above that if we wanted to buy apples, we would pay 1.13 per pound when buying under 3 pounds. If we were buying 8 pounds or more, we would be spending 0.80 per pound.

## Storage Options

Tabular Lookup data can be stored in Sitecore in a few different ways.

1. **Sitecore Content Tree**: To store data in a Sitecore content tree, a folder would be created to hold the data for the table. Child items would be created to represent each row in the table. In this case, an item would exist for each fruit or vegetable. Additional fields on the item would represent the columns. In this case, we would have fields for each weight range.  ![Sitecore Content Tree](/images/storing-tabular-lookup-data-in-sitecore/StoringTabularLookupDataInSitecore_ContentTree.png) While this approach works, it may become difficult to maintain if there are lots of rows and columns that all need to be updated periodically.
1. **Custom Database Table**: A second approach would be to create an additional SQL database with tables to store the lookup data. There are several challenges with this approach, the primary challenge being the need to maintain and manage an additional database. To improve the usability of managing the data stored in the database, we may want to create content management screens for creation, updating and deleting the data. This feels like a lot of work.
1. **Spreadsheet**: Another option would be to manage the lookup data in a spreadsheet. The spreadsheet could be uploaded to Sitecore in the Media Library. There are already tools out there to manage large spreadsheets worth of data, and we could utilize those tools to manage the data. If the spreadsheet data is saved off as a .csv file, we could upload this into Sitecore and very easily read the data for use within the application. ![Spreadsheet](/images/storing-tabular-lookup-data-in-sitecore/StoringTabularLookupDataInSitecore_Spreadsheet.png)  In several cases, I have found it easiest to manage tabular lookup data in spreadsheets and upload those spreadsheets into Sitecore. This is especially true with large amounts of data that need to be updated periodically. Clients are generally already familiar with the use of the spreadsheets, and, in some cases, the original data was already being managed within spreadsheets. Using and uploading the original spreadsheets not only saves time, but it also lowers the learning curve for the end user.

## Implementation

We’ve decided the best approach would be to implement the spreadsheet approach for storing tabular lookup data in Sitecore. The only question left to ask is: how?

### Create the Spreadsheet

The first thing to do is create a spreadsheet with the data you want to use within Sitecore. This can either be in an already existing spreadsheet or one that will be created from scratch. Use your favorite editor for this. Once you have created the spreadsheet, save the data as a .csv format. The .csv format saves the spreadsheet as a text file. The columns are delimited by commas and the rows are separated by new line characters. Saving the file in this format makes it easy to parse the data programatically.

> Communicating the format of the Spreadsheet to end users is important. We will be programatically parsing the spreadsheet and will be expecting specific columns and rows. If an end user changes the format of the spreadsheet, it could break the parsing logic.

### Sitecore Content

We now need to save the **.csv** spreadsheet in Sitecore. The spreadsheet should be uploaded to the media library within Sitecore. You can organize where the spreadsheet is to be saved via folders if desired. Typically you would place the spreadsheet under the **Files** folder.

We can either reference the spreadsheet directly in the media library based on the id and/or path or we can create a separate content item that refers to the spreadsheet in the media library through a File field. I prefer to create a separate content item that will refer to the media item. Using a separate content item with a File field allows the content administrator to choose whatever spreadsheet they want to use without us hard-coding the path to the media item (ie. spreadsheet).

> The way you refer to the Media Item that contains the spreadsheet should really be based on what makes sense for the implementation. In this case, I am using a content item with a file field that points to the Media Item. This approach is useful if the content item is the current item and the data in the spreadsheet is required for the rendering. The content item could also be stored outside of the website and hold general settings used throughout the app. A hard-coded/configurable reference to this item would need to be made. Alternatively, rendering parameters could be used to point to the media item.

The referenced media item could be loaded as follows:

``` csharp
Sitecore.Data.Fields.FileField file = item.Fields[<FieldName>];
if (file != null)
{
  var mediaItem = file.MediaItem;
  ...
}
```

Once a reference to the MediaItem exists, we need to load the file content so we can parse it. The easiest way to parse a .csv file is to use something already written to do that. There is a **“CsvHelper”** on **NuGet** that handles parsing csv files. We can use this package to do so: [http://www.nuget.org/packages/CsvHelper/](http://www.nuget.org/packages/CsvHelper/). Here is an example of how to parse the CSV file contained in the media item: (continuing the example from above)

``` csharp
public static List<Price> ReadPrices(MediaItem mediaItem)
{
  var prices = new List<Price>();

  using (var stream = mediaItem.GetMediaStream())
  {
    using (TextReader reader = new StreamReader(stream))
    {
      var csv = new CsvReader(reader);
      while (csv.Read())
      {
        var price = new Price();
        price.Product = csv.GetField<string>("Product");
        price.PerLb = csv.GetField<string>("per lb");
        price.Per3Lb = csv.GetField<string>("3+ lb");
        price.Per5Lb = csv.GetField<string>("5+ lb");
        price.Per8Lb = csv.GetField<string>("8+ lb");

        prices.Add(price);
      }
      return prices;
    }
  }
}
```

The CsvReader makes it really easy to read data out of a csv file based on the column name. That data can then be mapped to properties on a class. We can then perform any lookup we would like based on the returned list of prices.

From a performance perspective, it would be beneficial to cache the results of the lookup. The lookup not only has to load the spreadsheet from Sitecore, but then it has to parse the data. Caching would be beneficial in this case.

Each Sitecore Item has a revision associated with it. This includes MediaItems. Each time an item is updated, the revision changes. We can use the revision as a way to tell if the cache needs to be invalidated when a new Spreadsheet is uploaded. We can create a wrapper around the method above that performs the caching.

``` csharp
private static List<Price> _prices;
private static string _pricesRevision;
private static readonly object _lock = new object();

public static List<Price> GetPrices(MediaItem mediaItem)
{
  var revision = mediaItem.Statistics.Revision;

  if (revision != _pricesRevision)
  {
    lock (_lock)
    {
      if (revision != _pricesRevision)
      {
        _prices = ReadPrices(mediaItem);
        _pricesRevision = revision;
      }
    }
  }
  return _prices;
}
```

> The code examples above provide a sample of how to load a referenced MediaItem, parse the csv content, and cache the data results. You may want to implement the above as a singleton scoped object or cache using an alternate method.

## Conclusion

There are various ways to store tabular lookup data within Sitecore. Storing the data as a spreadsheet within the Media Library is a quick and easy way to store this type of data.