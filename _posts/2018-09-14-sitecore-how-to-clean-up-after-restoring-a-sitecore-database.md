---
layout: post
title: "Sitecore How-To: Clean Up After Restoring a Sitecore Database"
date:   2018-09-14 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Eric Carron and Mike Skutta
excerpt: After you import/restore and Sitecore database, you will want to clean up some SQL tables.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

After you import/restore a Sitecore database, you will want to clean up some SQL tables.

> Your Lucene indexing may not work correctly until this step is completed.

## Step-by-step guide

1. Execute the following SQL statement on each of the Sitecore databases (**Core**, **Master**, **Web**):

    ```sql
    TRUNCATE TABLE [dbo].[EventQueue];
    TRUNCATE TABLE [dbo].[PublishQueue];
    TRUNCATE TABLE [dbo].[Properties];
    ```
2. Execute the following SQL statement on the **Web** database. It is helpful to have history in the **Master** database.

    ```sql
    TRUNCATE TABLE [dbo].[History];
    ```

> NOTE: Some modules may add data to the properties table.  Removing this data may have repercussions on those modules.  