---
layout: post
title: "Sitecore Data Modeling Approach for Integrations with Language Fallback"
date:   2016-11-14 00:00:00 -0500
categories: sitecore
tags: sitecore data
author: Mike Skutta
excerpt: We have several clients that integrate or sync data from their HR system into Sitecore.  This typically consists of “people data.”  These people appear on the website with their respective biographies. The data is brought over to Sitecore on a recurring basis.  While the HR system is considered the authoritative source for the people data, sometimes the marketing department requires the ability to override the data in Sitecore.  In some cases, the data from the HR system is not the data they want to display on the website. How do you support this within Sitecore?  There are various ways to setup the data templates and structure the data.  One way we have found that works well and does not require any additional logic is using language fallback. 
---

* content
{:toc}

## Overview

We have several clients that integrate or sync data from their HR system into Sitecore. This typically consists of "people data." These people appear on the website with their respective biographies. The data is brought over to Sitecore on a recurring basis. While the HR system is considered the authoritative source for the people data, sometimes the marketing department requires the ability to override the data in Sitecore. In some cases, the data from the HR system is not the data they want to display on the website.

How do you support this within Sitecore? There are various ways to setup the data templates and structure the data. One way we have found that works well and does not require any additional logic is using language fallback.

## Approach

Starting from Sitecore 8.1, language fallback is supported out of the box. Language fallback is best explained in this article: [Language Fallback](https://doc.sitecore.net/sitecore_experience_platform/setting_up__maintaining/language_fallback/language_fallback) The key take away is, "If a version does not exist in a given language, language fallback activates, and the item or field value is displayed in the fallback language instead."

We can use the language fallback to support the data sync and ability to override data, and then take the approach of setting up a base &ldquo;integration&rdquo; language. When data is brought in through a data sync, the data is saved in the integration language version of the item. Users are not allowed to edit the integration language version of the item. Only the data sync can modify the data in the integration language version.

The other languages fallback to the integration language. For example, English falls back to the integration language version. The public facing site does not directly reference the integration language.& The site rather references the languages that fallback to the integration language.

By default, the English language version does not have the fields filled out that are populated by the data sync. This allows the English language version of the item to automatically use the field values that come across as part of the integration. If the marketing team wanted to override the values from the integration, all they need to do is set the value of the field in the English language version of the item.& This automatically overrides the value from the data sync. If they want to revert back to the value from the data sync, all they need to do is reset the field.

**The benefits of using this approach are as follows:**

1. Language fallback is natively supported in Sitecore.
1. Developers building the front end of the site do not need to be aware of the data coming from the integration or if it has been overridden.
1. Glass Mapper and most other ORMs naturally works with this approach.
1. Lucene and other search technologies naturally work with this approach.

Here is a diagram depicting the approach:

![Data Modeling in Sitecore](/images/sitecore-data-modeling-approach-for-integrations-with-language-fallback/data-modeling_1.png)

## Implementation

Sitecore uses the languages/cultures registered on the server.&nbsp; If you want to create a unique language to represent the integration language, you can optionally create a new system.&nbsp; Alternatively, you can use an existing language.

To create a custom system language, you can use the following code:

*RegisterCustomCulture.cs*
``` csharp
string culture = "int";
string name = "Integration";
 
//Create Culture & Region Info with an existing Language Culture (Eg: en-GB)
CultureInfo cultureInfo = new CultureInfo("en-US");
RegionInfo regionInfo = new RegionInfo(cultureInfo.Name);
CultureAndRegionInfoBuilder cultureAndRegionInfoBuilder = new CultureAndRegionInfoBuilder(culture, CultureAndRegionModifiers.None);
cultureAndRegionInfoBuilder.LoadDataFromCultureInfo(cultureInfo);
cultureAndRegionInfoBuilder.LoadDataFromRegionInfo(regionInfo);
cultureAndRegionInfoBuilder.CultureEnglishName = cultureAndRegionInfoBuilder.CultureNativeName = name;
cultureAndRegionInfoBuilder.Register();
```

We can then register the integration language in Sitecore:

![Sitecore implementation](/images/sitecore-data-modeling-approach-for-integrations-with-language-fallback/data-modeling_2.png)

For the other languages, set the Language Fallback appropriately:

![Data modeling in sitecore](/images/sitecore-data-modeling-approach-for-integrations-with-language-fallback/data-modeling_3.png)

For the Fields that are populated via the data sync, they need to be setup as NOT Shared and NOT Unversioned. Enable field level fallback also needs to be enabled.

![Data modeling Approach for Integrations with Language Fallback](/images/sitecore-data-modeling-approach-for-integrations-with-language-fallback/data-modeling_4.png)

You can optionally lock down editing content in the integration language via security.

After these changes are made, you should be able to load data into the integration language and allow the other languages to override.

One thing we need to make sure never happens is publicly exposing the integration language version of the item. We don't want the integration version to be linked to or indexed by Google. The language versions that fallback to the integration language are the versions that should be exposed. Logic can be added to the &lt;httpRequestBegin&gt; pipeline to prevent this from happening.

## Conclusion

This Data Modeling approach has served us well when it comes to data integrations and data syncs. There are definitely other approaches out there.
