---
layout: post
title: "Sitecore How-To: Link to a Sitecore Item in the Content Tree that Will Auto Expand to that Item"
date:   2018-10-04 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Alex Hawley and Mike Skutta
excerpt: If you have the ID of an item in the Sitecore tree then you can create a link that will auto expand the tree to that item for other users to reference.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

If you have the ID of an item in the Sitecore tree, then you can create a link that will auto expand the tree to that item for other users to reference.

## Step-by-step guide

1. Find the item in the content tree that you want to link to.
1. Copy the item ID in the content tree.
1. Create a url that matches the format:
**<sitename>/sitecore/shell/Applications/Content%20Editor?id=<item id (no brackets)>&fo=<item id (no brackets)>&sc_content=<database name (master or web)>**
1. When a user clicks the link, they will be sent to the item in the content tree.