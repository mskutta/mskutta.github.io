---
layout: post
title: "Sitecore How-To: Force a Resync of TDS Items"
date:   2020-11-14 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: When syncing TDS items with Sitecore, TDS will not always give you the Update Project option if there are differences.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

 When syncing TDS items with Sitecore, TDS will not always give you the `Update Project` option if there are differences. Some standard fields like `__Created` will not trigger the `Update Project` option, and you will see that often if a site has been upgraded. You may want to have TDS sync everything to make sure your project matches 100%. Here is how you do that.

## Step-by-step guide

### Option 1 - Update TDS Sync Settings

There are global settings available in Visual Studio that can change which fields TDS will ignore during the sync process.  Since this is global, you may want to change these and then reset so that other projects aren’t impacted, but it is up to you.

**To get to the settings:**

1. From the main menu bar, go to `Tools > Options`
1. In the Options window, either scroll to *“TDS Options”* or search for it in the window and select the *“Sync Window Fields”* settings.
1. From this window, you can remove any of the default fields and click OK.
1. Perform a TDS sync to check for changes.
1. Optionally, you can go back to the *“Sync Window Fields”* settings and click the *“Reset”* button to revert back to the default fields.

### Option 2 - Clean Up TDS Items

This process will remove some data from the TDS `.item` files (like versions), but leaves them in a valid state. TDS will then see these items as missing data and will give you the `Update Project` option.

1. Create a TrimItemFiles.ps1 file from the script below.
1. Open PowerShell and navigate to the directory where you saved the script.
1. Execute the script with a *Path* parameter that specifies the root folder. The script will iterate through the folders and update any *.item* file it finds. 
    ``` shell
    .\TrimItemFiles.ps1 -Path C:\Projects\cla-sitecore\src\TDS.Core\sitecore\
    ```
1. Go back to your TDS projects and run a Sync with Sitecore and sync your items.


TrimItemFiles.ps1
``` powershell
param (
    [Parameter(Mandatory=$true)][string]$path
 )

Get-ChildItem -Path $path -Include *.item -Recurse | ForEach-Object {
    Write-Host $_.FullName
    $text = [IO.File]::ReadAllText($_.FullName)
    $lines = @()
    $add = $true

    $text -split "`r`n|`n" | ForEach {
        if ($add -and $_.Equals("----version----")) {
            $add = $false
            $lines += ""
        }

        if ($add) {
            $lines += $_
        }
    }

    $output = ($lines -join "`r`n") -replace "(`r`n){2,}$", "`r`n"

    [IO.File]::WriteAllText($_.FullName, $output)
}
```