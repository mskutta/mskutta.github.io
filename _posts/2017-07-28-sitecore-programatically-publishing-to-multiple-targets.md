---
layout: post
title: "Sitecore Programatically Publishing to Multiple Targets"
date:   2017-07-28 00:00:00 -0500
categories: sitecore
tags: sitecore publish target
author: Mike Skutta
image:
    url: /images/sitecore-programatically-publishing-to-multiple-targets/content-tree.png
    height: 768
    width: 1024
excerpt: I run into various occasions where I need to programmatically publish an Sitecore item.  One thing you don't want to forget about are multiple publishing targets.  You may have more than one target due to hosting content delivery servers in different geographic regions with separate web databases.  Instead of publishing to a fixed or hard-coded database, you should use the configured publishing targets to determine the databases to publish to.
---

* content
{:toc}

## Overview

I run into various occasions where I need to programmatically publish an Sitecore item.  One thing you don't want to forget about are multiple publishing targets.  You may have more than one target due to hosting content delivery servers in different geographic regions with separate web databases.  Instead of publishing to a fixed or hard-coded database, you should use the configured publishing targets to determine the databases to publish to.

![Content Tree](/images/sitecore-programatically-publishing-to-multiple-targets/content-tree.png)

## Example

Below is a code example that I use for my own reference to publish to ALL publishing targets.

``` csharp
// Get all publishing targets
var publishingTargets = Sitecore.Publishing.PublishManager.GetPublishingTargets(item.Database);

// Loop through each target, determine the database, and publish
foreach(var publishingTarget in publishingTargets)
{
    // Find the target database name, move to the next publishing target if it is empty.
    var targetDatabaseName = publishingTarget["Target database"];
    if (string.IsNullOrEmpty(targetDatabaseName))
        continue;

    // Get the target database, if missing skip
    var targetDatabase = Sitecore.Configuration.Factory.GetDatabase(targetDatabaseName);
    if (targetDatabase == null)
        continue;

    // Setup publishing options based on your need
    var publishOptions = new Sitecore.Publishing.PublishOptions(
                item.Database,
                targetDatabase,
                Sitecore.Publishing.PublishMode.Smart,
                item.Language,
                System.DateTime.Now);

    // Perform the actual publish
    var publisher = new Sitecore.Publishing.Publisher(publishOptions);
    publisher.Options.RootItem = item;
    publisher.Options.Deep = false;
    publisher.Publish();
}

```

In the above example, **item** is assumed to be coming from the master (Content Management) database.

