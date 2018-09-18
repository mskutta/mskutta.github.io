---
layout: post
title: "Sitecore How-To: Configure TDS for Local Development"
date:   2018-09-19 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta 
excerpt: When you want to configure the TDS projects within a Sitecore solution for local development.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge.  I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When you want to configure the TDS projects within a Sitecore solution for local development.

> We typically use the **TDS.Master** project for file deployment. Some projects have a dedicated TDS project for file deployment named **TDS.Files**. **TDS.Core** is not used for file deployment.  **TDS.Master** and **TDS.Core** is also used to manage item files for the respective database.

> Checking the Edit user specific configuration setting will save your settings to the .user file, which is not in source control.

## Step-by-step guide

1. Make sure your build configuration is **Debug**.
1. Configure the **TDS.Master** project
    1. Right-click the **TDS.Master** project and click Properties.
    1. Click the Build tab.
    1. Check Edit user specific configuration (.user file)
    1. Enter your Sitecore Web Url (i.e. http://mysitecoresite)
    1. Enter the Sitecore Deploy Folder (i.e. C:\projects\mysitecoresite\Website)
    1. Make sure Disable file deployment **is _NOT_ checked**.
    1. Make sure Install Sitecore Connector is checked.
    1. Save
    1. Optional, if the Sitecore site is up and running, click Test.
1. Configure the **TDS.Core** project
    1. Right-click the **TDS.Core** project and click Properties.
    1. Click the Build tab.
    1. Check Edit user specific configuration (.user file)
    1. Enter your Sitecore Web Url (i.e. http://mysitecoresite)
    1. Enter the Sitecore Deploy Folder (i.e. C:\projects\mysitecoresite\Website)
    1. Make sure Disable file deployment is checked.
    1. Make sure Install Sitecore Connector is checked.
    1. Save.
    1. Optional, if the Sitecore site is up and running, click Test.
1. Make sure the Sitecore Access Guid is the same value for **TDS.Core** and **TDS.Master**.