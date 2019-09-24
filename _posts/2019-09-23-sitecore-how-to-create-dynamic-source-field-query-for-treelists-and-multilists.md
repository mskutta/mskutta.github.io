---
layout: post
title: "Sitecore How-To: Create Dynamic Source Field Query for Treelists and Multilists in Sitecore"
date:   2019-09-23 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Alex Pershteyn and Mike Skutta
excerpt: Treelist and Multilist fields in Sitecore provide a way of specifying what is displayed and what is selectable in a selection pane of the field in Source field. Out-of-the-box you can use sitecore queries to populate these fields. Additionally, you can create a fully dynamic list of available items in Source field where you can implement custom client-specific business logic for populating selectable values.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Treelist and Multilist fields in Sitecore provide a way of specifying what is displayed and what is selectable in a selection pane of the field in Source field. Out-of-the-box you can use sitecore queries to populate these fields. For example, if you need a field that allows you to select Highlights your Source property may look something like:

```xml
DataSource=/sitecore/content/sites/main/home/highlights/
```

You could further constrain the elements in the selection pane by using the following parameters:

* IncludeTemplatesForSelection: Users can only select items based on this comma-separated list of data template names.
* ExcludeTemplatesForSelection: Users cannot select items based on this comma-separated list of data template names.
* IncludeTemplatesForDisplay:Users can view items based on this comma-separated list of data template names and IDs.
* ExcludeTemplatesForDisplay: Users cannot navigate items based on this comma-separated list of data template names.
* IncludeItemsForDisplays: Users can navigate items based on this comma-separated list of item names and IDs.
* ExcludeItemsForDisplay:  Users cannot navigate items based on this comma-separated list of item names and IDs.
* AllowMultipleSelection: If yes, users can select the same item more than once.

More information about these parameters can be found in [this blog](http://zacharykniebel.com/blog/sitecore/2014/june/26/constraining-the-sitecore-7-multilist-and-treelist-fields-with-and-without-search) post or in [Data Definition Cookbook (page 12-13)](https://sdn.sitecore.net/upload/sitecore6/60/data_definition_cookbook_sc62-a4.pdf).

Additionally, you can create a fully dynamic list of available items in Source field where you can implement custom client-specific business logic for populating selectable values.

## Step-by-step guide

1. Include Sitecore.Buckets.dll as a reference to in your project
1. Create a new class that implements Sitecore.Buckets.FieldTypes.IDataSource interface and returns a custom array of Items. You could use Lucene here but will need to convert search results back into Sitecore items

    ```c#
    using Sitecore.Buckets.FieldTypes;
    using Sitecore.Data.Items;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    
    namespace Website.Logic.CustomDataSources
    {
        public class RelatedInsightsDataSource : IDataSource
        {
            public Item[] ListQuery(Item item)
            {
                List<Item> dataSource = new List<Item>();
                //add your custom logic here that gets a list of source items for your multilist
    
                return dataSource.ToArray();
    
            }
        }
    }
    ```
1. Update a source field for the field you are updating with your own class: code:Website.Logic.CustomDataSources.RelatedInsightsDataSource, Website

      ![Source](/images/sitecore-how-to-create-dynamic-source-field-query-for-treelists-and-multilists/source.png "Source")