---
layout: post
title: "Site Context, Computed Fields and Live Mode in Sitecore 8.1"
date:   2016-06-08 00:00:00 -0500
categories: sitecore
tags: sitecore context computed-fields live-mode
author: Mike Skutta
excerpt: We ran into an issue with code that worked in Sitecore 8.0 but not in Sitecore 8.1.  It took us a while to determine the root cause of the issue and how to fix it, so I thought I would share what we found to help anyone else that might run into this issue.
---

* content
{:toc}

## Overview

We ran into an issue with code that worked in Sitecore 8.0 but not in Sitecore 8.1.  It took us a while to determine the root cause of the issue and how to fix it, so I thought I would share what we found to help anyone else that might run into this issue.

## Symptoms

Rebuilding indexes either though the control panel or on Azure role startup will hang at random locations.  This will prevent the role from starting up properly, and the index is stuck in an invalid state.

This hang only occurs when you have the parallel indexing enabled and you have live mode enabled on Sitecore 8.1.

## Diagnosis

We started to add ** Sitecore.Context.SetActiveSite("website"); ** to our computed fields due to null exception when generating links during index rebuild on CM in Azure.  This was done because the site context changed in 8.1 when you rebuild the index.  The issue with this fix was that it would create an additional null condition when an item was marked as “never publish.”  When an item was marked as “never publish,” it would cause the exception within Sitecore indexing process to hang indefinitely.

## Solution

The solution was to use the SiteContextSwitcher that implements the IDisposable interface.

``` csharp
using (new SiteContextSwitcher(SiteContext.GetSite("website")))
{
    // do something on the new site context
}
```

For computed fields, this code should be used to wrap logic that requires a site context.

In our case, we were using the link manager that required a site context within computed fields.

## References

Here is a good article that explains the proper way to switch Sitecore’s SiteContext: [http://rajchel.pl/2014/12/switching-sitecore-sitecontext/](http://rajchel.pl/2014/12/switching-sitecore-sitecontext/)