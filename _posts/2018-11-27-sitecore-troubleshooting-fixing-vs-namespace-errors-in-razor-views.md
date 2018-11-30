---
layout: post
title: "Sitecore Troubleshooting: Fixing VS Namespace Errors in Razor Views"
date:   2018-11-27 00:00:00 -0500
categories: sitecore
tags: sitecore-troubleshooting
author: Kyle Mattimore and Mike Skutta
excerpt: The type or namespace name Mvc does not exist in the namespace Sitecore (are you missing an assembly reference?).
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore troubleshooting articles. These articles are meant to be quick guides to troubleshoot various issues within Sitecore. The troubleshooting articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## Problem

When editing razor views in Visual Studio, you see many errors like

```
The type or namespace name 'Mvc' does not exist in the namespace 'Sitecore' (are you missing an assembly reference?)
```

or

```
The type or namespace name 'Data' does not exist in the namespace 'Sitecore' (are you missing an assembly reference?)
```

for any view file you have open at the time. This can happen even if the namespaces are referenced in the web.config in the Views folder. They are annoying and break intellisense but don't actually prevent build or cause any runtime errors. It seems to happen more on later versions of visual studio (2015 / 2017).

## Solution

Find which (likely Sitecore) DLL that contain the namespace (e.g. Sitecore.Kernel.dll for Sitecore.Data). Find the reference to that DLL in the Website project's references and look at the 'copy local' property.

![Copy Local Property](/images/sitecore-troubleshooting-fixing-vs-namespace-errors-in-razor-views/image2017-10-20_10-8-50.png)

'Copy Local' controls whether the DLL is copied to the bin folder in the code (not the web root). For whatever reason, VS has problems with namespaces that aren't in the local bin. 

If it is false, set it to true, then click 'save all'. If it is already true.... there is some other problem.

> Version Matching
>
> At this point you need to be sure the version you are referencing matches the version that comes included in the Sitecore installer. If the versions match, you can skip the next step. If they don't, you could risk deploying the wrong version of that DLL and breaking sitecore in subtle, terrible ways. 

Now we have to verify that this doesn't cause this DLL to be unnecessarily deployed out to the actual web root. This is handled by TDS. Open the TDS project 'General' tab in properties. You should see an exclude list for Sitecore DLLs, either individual or catching all with a wildcard: 

![Copy Local Property](/images/sitecore-troubleshooting-fixing-vs-namespace-errors-in-razor-views/image2017-10-20_10-22-11.png)

You don't want these DLLs copied to your web root because the Sitecore install (both locally and in staging/prod deployments) puts them there. If you don't see your DLL on this list, add it, assuming that it is a Sitecore DLL that was indeed put there by the installer or another installed Sitecore module. 

**Build the project** after making and saving these changes. If you're paranoid, go to your local website bin and temporarily rename the dll, e.g. to Sitecore.Kernel.dll.old. Then build the project again. If the DLL doesn't re-appear in your website bin you're good. 

**Close and re-open visual studio** and the reference errors should be gone. 