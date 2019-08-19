---
layout: post
title: "Sitecore How-To: Add Sitecore NuGet Packages to Your Project"
date:   2019-08-19 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: When you want to reference Sitecore binaries in your Visual Studio project, use Sitecore's NuGet Feed.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

When you want to reference Sitecore binaries in your Visual Studio project, use Sitecore's NuGet Feed.

> Starting with Sitecore 9.1, Sitecore changed their NuGet package strategy so the version name is major.minor.update (instead of major.minor.rev) and they no longer create the NoReferences package variations. If you only want to reference the package and not add its dependencies, make sure to specify the Ignore Dependencies option. 

## Step-by-step guide

1. Make sure you add Sitecore's NuGet Feed to Visual Studio: [Sitecore How-To: Add Sitecore's NuGet Repository To Visual Studio](/2018/09/13/sitecore-how-to-add-sitecores-nuget-repository-to-visual-studio/)
1. Using the NuGet UI within Visual Studio:
    1. Browse for the package you want to add (Sitecore.Kernel) and make sure Package source (drop down in top right corner) is either All or Sitecore NuGet (the source you added in step 1).
    1. See the above note about NoReferences packages and the Ignore Dependencies option.
    1. Select the Sitecore version you are working with and install.
1. Using the NuGet Package Manager Console: 
   ```powershell
   Install-Package -ProjectName Website -IgnoreDependencies -ID Sitecore.Kernel -Version 9.1.1
   ```
   or, if you want all dependencies
   ```powershell
   Install-Package -ProjectName Website -ID Sitecore.Kernel -Version 9.1.1
   ```

> Installing dependencies of a Sitecore package will install many packages and is most likely not desired. The only exception is for your main Website project, where you may want it to install the correct packages for MVC, Lucene, etc.