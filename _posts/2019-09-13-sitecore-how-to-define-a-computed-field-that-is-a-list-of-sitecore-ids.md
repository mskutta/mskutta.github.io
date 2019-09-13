---
layout: post
title: "Sitecore How-To: Define a Computed Field That is a List of Sitecore IDs"
date:   2019-09-13 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: When you have a computed field in a Lucene index that is a list of Sitecore IDs, there are a few things you can do to make your life easier.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When you have a computed field in a Lucene index that is a list of Sitecore IDs, there are a few things you can do to make your life easier. 

## Step-by-step guide

1. Create the class that defines your computed field. Make sure it returns a collection of Sitecore.Data.ID.

    ```c#
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.ComputedFields;
    using Sitecore.Data;
    using Sitecore.Data.Items;
    using System.Collections.Generic;
    using System.Linq;
    
    namespace Website.ComputedFields
    {
        public class RelatedItemsComputedField : AbstractComputedIndexField
        {
            public override object ComputeFieldValue(IIndexable indexable)
            {
                Item item = indexable as SitecoreIndexableItem;
    
                if (item == null)
                {
                    return null;
                }
    
                List<ID> ids = new List<ID>();
    
                // TODO: populate ids
    
                if (ids.Any(x => !ID.IsNullOrEmpty(x)))
                {
                    return ids.Where(x => !ID.IsNullOrEmpty(x)).Distinct().ToList();
                }
    
                return null;
            }
        }
    }
    ```
1. Add the computed field to your index configuration.
    ```xml
    <configuration xmlns:patch="http://www.sitecore.net/xmlconfig/" xmlns:search="http://www.sitecore.net/xmlconfig/search/" xmlns:role="http://www.sitecore.net/xmlconfig/role/">
      <sitecore role:require="Standalone or ContentManagement or ContentDelivery" search:require="lucene">
        <contentSearch>
          <indexConfigurations>
            <oneNorthLuceneIndexConfiguration ref="contentSearch/indexConfigurations/defaultLuceneIndexConfiguration">
    
              <initializeOnAdd>true</initializeOnAdd>
    
              <fieldMap ref="contentSearch/indexConfigurations/defaultLuceneIndexConfiguration/fieldMap">
                <fieldNames hint="raw:AddFieldByFieldName">
                  <field fieldName="relateditems" storageType="NO" indexType="TOKENIZED" vectorType="NO" boost="1f" type="System.String" settingType="Sitecore.ContentSearch.LuceneProvider.LuceneSearchFieldConfiguration, Sitecore.ContentSearch.LuceneProvider" >
                    <analyzer type="Sitecore.ContentSearch.LuceneProvider.Analyzers.LowerCaseKeywordAnalyzer, Sitecore.ContentSearch.LuceneProvider" />
                  </field>
                </fieldNames>
              </fieldMap>
    
              <documentOptions type="Sitecore.ContentSearch.LuceneProvider.LuceneDocumentBuilderOptions, Sitecore.ContentSearch.LuceneProvider">
    
                <fields hint="raw:AddComputedIndexField">
                  <field fieldName="relateditems">Website.ComputedFields.RelatedItemsComputedField, Website</field>
                </fields>
    
              </documentOptions>
    
            </oneNorthLuceneIndexConfiguration>
          </indexConfigurations>
        </contentSearch>
      </sitecore>
    </configuration>
    ```
1. Create the class that maps to your index search results. For your computed field, make it IEnumerable<ID> and use the TypeConverter attribute.
    ```c#
    using Sitecore.ContentSearch;
    using Sitecore.ContentSearch.Converters;
    using Sitecore.ContentSearch.SearchTypes;
    using Sitecore.Data;
    using System.Collections.Generic;
    using System.ComponentModel;
    using System.Runtime.Serialization;
    
    namespace Website.Search.Models
    {
        public class MySearchResultItem : SearchResultItem
        {
            [DataMember]
            [IndexField("relateditems")]
            [TypeConverter(typeof(IndexFieldEnumerableConverter))]
            public IEnumerable<ID> RelatedItems { get; set; }
        }
    }
    ```
1. Use your computed field in Linq queries.
    ```c#
    var query = PredicateBuilder.True<MySearchResultItem>();
    query = query.And(x => x.RelatedItems.Contains(someSitecoreID));
    ```
> Sitecore will take that collection of Sitecore IDs and index them as a collection of ShortIDs (no brackets, no hyphens, lowercase).

> When using Linq, Sitecore will generate a Lucene query similar to: +relateditems:8d9f908e905248568f1701baaeac2b19