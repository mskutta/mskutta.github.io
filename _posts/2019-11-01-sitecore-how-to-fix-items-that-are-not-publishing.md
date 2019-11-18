---
layout: post
title: "Sitecore How-To: Fix Items that are not Publishing"
date:   2019-11-01 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Eric Carron and Mike Skutta
excerpt: There are multiple requirements that need to be met for a Sitecore item to be published. Here are the various things that you can check for if your item is not being published.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

There are multiple requirements that need to be met for a Sitecore item to be published. Here are the various things that you can check for if your item is not being published.

## Step-by-step guide

* Verify that your item is in the final workflow state, if a workflow is specified. If no workflow is specific, then workflow state should be empty.
* Verify that your item is publishable, or if a particular version is not getting published, make sure the version is publishable.
* Make sure your publishing targets are set correctly. By default, no publishing targets should be selected, which means Sitecore will deploy to all publishing targets. View the raw value of the Publishing Targets field, as sometimes an old, non-existing publishing target is specified and the UI will not show it. If there are any values specified for Publishing Targets, the best way to clear it is to reset the field.
* Make sure your publishing targets are setup correctly. Typically we have one publishing target named Internet and the target database is web.
* Make sure your item name is valid. Sometimes we update the invalid characters list after items have been created.
* Make sure none of the parent items fail to publish.

Once you have verified all of these, you should be able to publish the item and it should show in the web database. If your item is in the web database and you are still unable to access the item via the web, check these items:

* Make sure your item name is valid. Sometimes we update the invalid characters list after items have been created and published. Item names with hyphens in particular usually cause issues.
* Make sure there aren't any Sitecore Aliases are interfering (see /sitecore/system/Aliases).