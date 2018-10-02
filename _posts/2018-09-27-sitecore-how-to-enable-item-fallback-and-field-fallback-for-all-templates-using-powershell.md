---
layout: post
title: "Sitecore How-To: Enable Item Fallback and Field Fallback for All Templates Using PowerShell"
date:   2018-09-27 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Rikin Shah and Mike Skutta
excerpt: For multilingual sites, you may find the need to enable item fallback and field fallback across all templates and fields.  Enabling item and field fallback removes the need to enter content for all items and fields when they don't need to be translated.  It can be very tedious to enable it in Sitecore by going to each template and field. You can use Sitecore PowerShell Extensions to run scripts to enable it for all templates.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

For multilingual sites, you may find the need to enable item fallback and field fallback across all templates and fields.  Enabling item and field fallback removes the need to enter content for all items and fields when they don't need to be translated.  It can be very tedious to enable it in Sitecore by going to each template and field. You can use Sitecore PowerShell Extensions to run scripts to enable it for all templates.

## Step-by-step guide

1. Install Powershell Module from https://marketplace.sitecore.net/en/Modules/Sitecore_PowerShell_console.aspx
1. Open Sitecore Powershell ISE in sitecore.
1. Set the ISE Context to master:\templates.
1. Execute the following scripts:

**Enable Field Fallback**
```powershell
Get-ChildItem -Recurse |
    Where-Object { $_.Template.Name -eq "Template field" } |
    Where-Object { $_["Enable Shared Language Fallback"] -ne "1" } |
    ForEach-Object {
        Write-Host "Updating Item" $_.Name
        $_.Editing.BeginEdit()
        $_["Enable Shared Language Fallback"] = "1"
        $_.Editing.EndEdit()
    }
```

**Enable Item Fallback**
```powershell
Get-ChildItem -Recurse |
    Where-Object { $_.Name -eq "__Standard Values" } |
    Where-Object { $_["__Enable item fallback"] -ne "1" } |
    ForEach-Object {
        Write-Host "Updating Item" $_.Paths.FullPath
        $_.Editing.BeginEdit()
        $_["__Enable item fallback"] = "1"
        $_.Editing.EndEdit()
    }
```