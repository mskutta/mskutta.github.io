---
layout: post
title: "Sitecore How-To: Disable Sitecore Publish Site"
date:   2020-11-09 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: There are various ways to publish the entire site in Sitecore and it is too often accidentally started by content editors. This is how you hide/disable all the different ways to publish the entire site.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

There are various ways to publish the entire site in Sitecore and it is too often accidentally started by content editors. This is how you hide/disable all the different ways to publish the entire site.

> Note: This method will hide/remove Publish Site for all users except administrators.

## Step-by-step guide

Deny read access to the `sitecore\Sitecore Client Users` role on the following items:

1. `/sitecore/system/Settings/Key Map/Ctrl-F9`
1. `/sitecore/content/Applications/Content Editor/Menues/Publish/Publish Site`
1. `/sitecore/content/Documents and settings/All users/Start menu/Left/Publish Site`

> Note: There are various ways you can hide these items, but using a deny read access to `sitecore\Sitecore Client Users` is the easiest way.