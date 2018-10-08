---
layout: post
title: "Sitecore How-To: Migrate from Field Fallback to Language Fallback using PowerShell"
date:   2018-10-05 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Before Sitecore provided the Language Fallback feature, you had to use Hedgehog's Field Fallback module from the Sitecore Marketplace. When upgrading to a Sitecore instance that has Language Fallback, you will want to migrate the settings from the Field Fallback module. Here is how to do that using Sitecore PowerShell Extensions.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Before Sitecore provided the Language Fallback feature, you had to use Hedgehog's Field Fallback module from the Sitecore Marketplace. When upgrading to a Sitecore instance that has Language Fallback, you will want to migrate the settings from the Field Fallback module. Here is how to do that using Sitecore PowerShell Extensions.

## Step-by-step guide

1. Install [Sitecore PowerShell Extensions](https://marketplace.sitecore.net/en/Modules/Sitecore_PowerShell_console.aspx) module from the Sitecore Marketplace.
1. Open Sitecore Powershell ISE in Sitecore.
1. Set the ISE Context to master:\templates.
1. Execute the following script to migrate the field fallback settings: 

    ```powershell
    Get-ChildItem -Recurse |
        Where-Object { $_.Template.Name -eq "Template field" } |
        Where-Object { $_.EnableLanguageFallback -eq "1" } |
        ForEach {
            Write-Host "Updating Item " $_.Paths.FullPath
            $_.Editing.BeginEdit()
            $_["Enable Shared Language Fallback"] = "1"
            $_.Editing.EndEdit()
              
            Reset-ItemField -Item $_ -Name "EnableLanguageFallback"
        }
    ```

1. (Optional) Execute the following script to remove Field Fallback fields: 

    ```powershell
    Remove-Item -Path "master:\system\Settings\Validation Rules\Field Rules\Fallback" -Permanently -Recurse
    Remove-Item -Path "master:\templates\System\Language\Fallback" -Permanently -Recurse
    Remove-Item -Path "master:\templates\System\Templates\Template field\Fallback" -Permanently -Recurse
    ```

> This migration will not enable Item Fallback. You will want to enable that manually per template or see How to Enable Item Fallback and Field Fallback for All Templates Using PowerShell.