---
layout: post
title: "Sitecore How-To: Allow Users to See All Items in Recycle Bin"
date:   2020-11-15 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Some clients have requested to be able to view all items in the recycle bin and not just the items they deleted. By default, Sitecore administrators are the only users who can see all items. In the past we had to create custom code to allow this, but starting in Sitecore 7.2 there is a policy that you can edit to allow this.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Some clients have requested to be able to view all items in the recycle bin and not just the items they deleted. By default, Sitecore administrators are the only users who can see all items. In the past we had to create custom code to allow this, but starting in Sitecore 7.2 there is a policy that you can edit to allow this.

## Step-by-step guide

1. In the `Core` database, navigate to `/sitecore/system/Settings/Security/Policies/Recycle Bin/Can See All Items`
1. Grant read access (Security → Assign) to any user/role that needs the ability to see all items in the recycle bin.
1. That’s it!