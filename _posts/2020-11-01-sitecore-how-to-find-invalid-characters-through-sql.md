---
layout: post
title: "Sitecore How-To: Find Invalid Characters through SQL"
date:   2020-11-01 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: David Guan and Mike Skutta
excerpt: While indexing with Solr, we ran into an issue that prevented indexing from continuing.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

While indexing with Solr, we ran into an issue that prevented indexing from continuing. An example error looks something like this:

```text
15152 15:18:20 WARN  [Index=onenorth_web_index] Crawler : AddRecursive DoItemAdd failed - {21F8A030-A782-404E-BDC8-5E2C95D1EFE8}
Exception: System.ArgumentException
Message: '', hexadecimal value 0x1F, is an invalid character.
Source: System.Xml
   at System.Xml.XmlEncodedRawTextWriter.WriteElementTextBlock(Char* pSrc, Char* pSrcEnd)
   at System.Xml.XmlEncodedRawTextWriter.WriteString(String text)
   at System.Xml.XmlWellFormedWriter.WriteString(String text)
   at System.Xml.Linq.ElementWriter.WriteElement(XElement e)
   at System.Xml.Linq.XElement.WriteTo(XmlWriter writer)
   at System.Xml.Linq.XNode.GetXmlString(SaveOptions o)
   at SolrNet.Commands.AddCommand`1.ConvertToXml()
   at SolrNet.Commands.AddCommand`1.Execute(ISolrConnection connection)
   at SolrNet.Impl.LowLevelSolrServer.SendAndParseHeader(ISolrCommand cmd)
   at Sitecore.ContentSearch.SolrProvider.SolrBatchUpdateContext.AddRange(IEnumerable`1 group, Int32 groupSize)
   at Sitecore.ContentSearch.SolrProvider.SolrBatchUpdateContext.Commit()
   at Sitecore.ContentSearch.CommitPolicyExecutor.IndexModified(IProviderUpdateContext context, Object document, IndexOperation operation)
   at Sitecore.ContentSearch.SolrProvider.SolrBatchUpdateContext.AddDocument(Object itemToAdd, IExecutionContext[] executionContexts)
   at Sitecore.ContentSearch.SolrProvider.SolrIndexOperations.ApplyPermissionsThenIndex(IProviderUpdateContext context, IIndexable version)
   at Sitecore.ContentSearch.SitecoreItemCrawler.DoAdd(IProviderUpdateContext context, SitecoreIndexableItem indexable)
   at Sitecore.ContentSearch.HierarchicalDataCrawler`1.CrawlItem(T indexable, IProviderUpdateContext context, CrawlState`1 state)
```

## Step-by-step guide

You might want to find and remove these characters manually.

1. Run this script on the master database of your site. This assumes you have already pulled down the databases from live. 
    ``` sql
    SELECT * FROM (
        SELECT ItemId, FieldId, Value FROM [REPLACE_WITH_MASTER_DB].[dbo].[SharedFields] 
        UNION
        SELECT ItemId, FieldId, Value FROM [REPLACE_WITH_MASTER_DB].[dbo].[UnversionedFields]
        UNION
        SELECT ItemId, FieldId, Value FROM [REPLACE_WITH_MASTER_DB].[dbo].[VersionedFields]
      ) A
    WHERE Value Like '%' + CHAR(0x01) + '%'
    --OR Value Like '%' + CHAR(0x08) + '%'
    --OR Value Like '%' + CHAR(0x10) + '%'
    --OR Value Like '%' + CHAR(0x12) + '%'
    --OR Value Like '%' + CHAR(0x1F) + '%'
    --Add others you might need as well, these are known ones
    ```
1. Alternatively, you might want to strip these out while indexing if it’s through a computed field so the client could still have the information in the backend.

> Note: You will still need to manually go into Sitecore to remove these items using this script.

