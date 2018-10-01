---
layout: post
title: "Sitecore How-To: Find Unicode Characters in Content"
date:   2018-10-01 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Sometimes unicode characters are accidentally added to content in Sitecore from data migration or from content editors copy-pasting content from some source. Depending on the unicode character, they can appear as random shapes on the front-end. Finding content with these unicode characters can be hard, but the following process will help you.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Sometimes unicode characters are accidentally added to content in Sitecore from data migration or from content editors copy-pasting content from some source. Depending on the unicode character, they can appear as random shapes on the front-end. Finding content with these unicode characters can be hard, but the following process will help you.

> Content in a non-English language will use unicode characters, so if the site has multiple languages, you will get a lot of results. To search for a specific unicode character, see the comments in the WHERE clause.

## Step-by-step guide

1. Run the following SQL statement on Master or Web. It will list all items and fields with a unicode character somewhere in its content. 

```sql
WITH [Versioned] AS
(
    SELECT
        [Id],
        [ItemId],
        [Language],
        [Version],
        [FieldId],
        [Value],
        [Created],
        [Updated],
        'Versioned' AS [FieldType]
    FROM
        [dbo].[VersionedFields] AS v
), [Unversioned] AS
(
    SELECT
        [Id],
        [ItemId],
        [Language],
        -1 AS [Version],
        [FieldId],
        [Value],
        [Created],
        [Updated],
        'Unversioned' AS [FieldType]
    FROM
        [dbo].[UnversionedFields] AS u
), [Shared] AS
(
    SELECT
        [Id],
        [ItemId],
        'N/A' AS [Language],
        -1 AS [Version],
        [FieldId],
        [Value],
        [Created],
        [Updated],
        'Shared' AS [FieldType]
    FROM
        [dbo].[SharedFields] AS s
), [AllFields] AS
(
    SELECT
        *
    FROM
        [Versioned]
         
    UNION ALL
 
    SELECT
        *
    FROM
        [Unversioned]
 
    UNION ALL
 
    SELECT
        *
    FROM
        [Shared]
), [ItemPath] AS
(
    SELECT
        p.[ID],
        CAST(p.[Name] AS nvarchar(500)) AS [Path],
        0 AS [Level]
    FROM
        [dbo].[Items] AS p
    WHERE
        [ParentID] = '00000000-0000-0000-0000-000000000000'
 
    UNION ALL
 
    SELECT
        c.[ID],
        CAST(p.[Path] + '/' + c.[Name] AS nvarchar(500)) AS [Path],
        p.[Level] + 1
    FROM
        [dbo].[Items] AS c
        INNER JOIN [ItemPath] AS p ON
            c.[ParentID] = p.[ID]
)
SELECT
    ip.[Path] AS [ItemPath],
    i.[Name] AS [ItemName],
    af.[Language],
    af.[Version],
    f.[Name] AS [FieldName],
    af.[FieldType],
    af.[Value] AS [FieldValue]
FROM
    [dbo].[Items] AS i
    INNER JOIN [AllFields] AS af ON
        af.[ItemId] = i.[ID]
    INNER JOIN [dbo].[Items] AS f ON
        f.[ID] = af.[FieldId]
    INNER JOIN [ItemPath] AS ip ON
        ip.[ID] = i.[ID]
WHERE
    af.[Value] != CAST(af.[Value] AS varchar(max))  -- all unicode characters
    --af.[Value] LIKE '%' + nchar(<DecimalValueHere>) + '%'  -- specific unicode character
ORDER BY
    ip.[Path],
    i.[Name],
    af.[Language],
    af.[Version],
    f.[Name]
```