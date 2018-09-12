---
layout: post
title: "Sitecore How-To: Add A Computed Index Field to Lucene"
date:   2018-09-12 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Computed index fields allow you to index values that are calculated during the indexing process.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Computed index fields allow you to index values that are calculated during the indexing process. 

> Implementation may vary depending on Sitecore version.  The below applies to Sitecore 7.0 to 9.0.

## Step-by-step guide

1. Create a class that derives from **IComputedIndexField**, which will contain the logic for the computed index field.

    ```c#
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.ComputedFields;
    using Sitecore.Data.Items;
    using Sitecore.Diagnostics;
    
    namespace Website.ComputedFields
    {
        public class MyComputedIndexField : IComputedIndexField
        {
            public string FieldName { get; set; }
    
            public string ReturnType { get; set; }
    
            public object ComputeFieldValue(IIndexable indexable)
            {
                Assert.ArgumentNotNull(indexable, "indexable");
                SitecoreIndexableItem scIndexable = indexable as SitecoreIndexableItem;
                Item item = (Item)scIndexable;
                Sitecore.Context.Database = item.Database;
    
                if (item != null)
                {
                    // This is for indexing the master database. We dont not want to index unpublishable items.
                    if (item.Publishing.IsPublishable(System.DateTime.Now, false))
                    {
                        // TODO: computed logic goes here. Return whatever value you want to be indexed.
                        // Caution: performance is critical. Keep your logic simple and stay away from ORMs if you can.
                    }
                }
    
                return null;
            }
        }
    }
    ```
1. Add the new computed index field to the configuration.
    1. Add the new field to the **raw:AddFieldByFieldName** section. This is where you will specify the storage type, index type, etc. The details of those settings are not covered in this article.
    
        ```xml
        <fieldNames hint="raw:AddFieldByFieldName">
          <field fieldName="myComputedIndexFieldName" storageType="YES" indexType="TOKENIZED" vectorType="NO" boost="1f" type="System.String" settingType="Sitecore.ContentSearch.LuceneProvider.LuceneSearchFieldConfiguration, Sitecore.ContentSearch.LuceneProvider" />
        </fieldNames>
        ```
    1. Add the new field to the fields hint="raw:AddComputedIndexField" section. This is where you will specify the class that contains the computed logic.

        ```xml
        <fields hint="raw:AddComputedIndexField">
          <field fieldName="myComputedIndexFieldName">Website.ComputedFields.MyComputedIndexField, Website</field>
        </fields>
        ```