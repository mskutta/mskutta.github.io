---
layout: post
title: "Sitecore Troubleshooting: Date Fields Off by One Day"
date:   2018-11-29 00:00:00 -0500
categories: sitecore
tags: sitecore-troubleshooting
author: Erik Carron and Mike Skutta
excerpt: There have been a few instances where date fields in Sitecore will be one day off when displayed on the frontend. Specifically, the backend will show one date, say 6-5-2018, but the frontend will render 6-4-2018.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore troubleshooting articles. These articles are meant to be quick guides to troubleshoot various issues within Sitecore. The troubleshooting articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## Problem

There have been a few instances where date fields in Sitecore will be one day off when displayed on the frontend. Specifically, the backend will show one date, say 6-5-2018, but the frontend will render 6-4-2018.

## Cause

The server time zone has been set to **GMT Standard Time**, which has daylight savings during summer months, and is not the same as UTC. During daylight savings, GMT Standard Time is one hour ahead of UTC. When a content editor picks a date in the backend, Sitecore converts that date (which is treated as a datetime field, but with time of midnight) from GMT Standard Time to UTC and then saves it to the database, so Sitecore will save 6-5-2018 GMT Standard Time as 6-4-2018 11:00 PM UTC (raw value in database: 20180604T230000Z). When this value is retrieved from the database, the date will be correct if converted back to GMT Standard Time, but if left as UTC, the date will be off by one day. Also, when using Glass Mapper, date fields are automatically converted back to the server time zone, but the Sitecore API will return date fields as UTC. This means your dates may be correct in some places, but not others, depending on the method you used to retrieve the data.

> DateTime fields are also affected by this, but are only off by one hour.

## Solution

Server time zone needs to be UTC and you may want to correct date fields that were saved when the server time zone was set to GMT Standard Time and was during daylight savings. If you decide to not correct these fields, some dates will remain to be a day off from what the content editor initially selected.

```xml
<setting name="ServerTimeZone" value="UTC"/>
```

To correct the data, you will need to run code that targets the incorrect fields after the server time zone has been switched to UTC.

Here is a SQL statement that might help identify the severity of the problem:

```sql
WITH [DateFields] AS
(
    SELECT
        f.[ItemId] AS [FieldId]
    FROM
        dbo.[SharedFields] AS f
    WHERE
        f.[FieldId] = 'AB162CC0-DC80-4ABF-8871-998EE5D7BA32'
        AND
        f.[Value] IN ('Date')
), [AllFields] AS
(
    SELECT [ItemId], [FieldId], [Value] FROM [dbo].[VersionedFields]
    UNION ALL
    SELECT [ItemId], [FieldId], [Value] FROM [dbo].[UnversionedFields]
    UNION ALL
    SELECT [ItemId], [FieldId], [Value] FROM [dbo].[SharedFields]
)
SELECT
    *
FROM
    [AllFields] AS f
WHERE
    EXISTS
    (
        SELECT NULL FROM DateFields AS df WHERE df.[FieldId] = f.[FieldId]
    )
    AND
    f.[Value] NOT LIKE '%T000000Z'
```