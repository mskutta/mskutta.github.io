---
layout: post
title: "Sitecore How-To: Fix TDS Item File Line Endings"
date:   2019-09-03 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: Team Development For Sitecore (TDS) uses item files to represent an item in Sitecore. There is a flaw in the design of these files where line endings are important. Depending on local configuration, git my automatically convert the line endings of these files, which can cause issues.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Team Development For Sitecore (TDS) uses item files to represent an item in Sitecore. There is a flaw in the design of these files where line endings are important. Depending on local configuration, git my automatically convert the line endings of these files, which can cause issues.

To prevent this from happening, the following needs to be done:

1. Prevent git from changing line endings in item files.
1. Correct any existing item files that have incorrect line ends in source control.

More details on this issue here: https://blogs.perficient.com/sitecore/2016/03/11/hedgehog-tds-field-content-does-not-match-content-length/

> TDS uses CRLF for line endings in item files, but fields may contain CRLF or LF. If git was auto fixing line endings, then item files will be normalized to LF in source control and that will cause the content length to be incorrect for certain fields. 

## Step-by-step guide

Prevent git from changing line endings in item files

> Execute the following steps in a clean repository with no pending changes.

1. Add the following to gitattributes. See [Git Attribute Templates](https://onenorthi.atlassian.net/wiki/pages/createpage.action?spaceKey=DOC&title=Git+Attribute+Templates&linkCreation=true&fromPageId=319815687) for a good example of what your gitattributes file should look like.

    ```
    *.item -text
    ```
1. Once you add this, you will want to undo any line ending normalization that git has already done then commit all this cleanup.
    ```sh
    rm .git/index
    git reset
    git status
    git add -u
    git add .gitattributes
    git commit -m "Normalize all the line endings"
    ```

Correct any existing item files that have incorrect line ends in source control

> Failing to sync items with richtext or multiline fields will result in invalid item files.

1. Make sure all item files have the CRLF line ending by downloading and running the following PowerShell script: 
    ```powershell
    param (
      [Parameter(Mandatory=$true)][string]$path 
    )
    Get-ChildItem -Path $path -Include *.item -Recurse | ForEach-Object {  
      ## If contains UNIX line endings, replace with Windows line endings
      if (Get-Content $_.FullName -Delimiter "`0" | Select-String "[^`r]`n") {        
        Write-Host $_.FullName        $text = [IO.File]::ReadAllText($_.FullName) -replace "(?<!\r)`n", "`r`n"
        [IO.File]::WriteAllText($_.FullName, $text)
      }
    }
    ```
1. Any items that have richtext content or multiline content should be re-sync with Sitecore to ensure correct line endings are in the item file.
1. Commit any pending changes that you have for item files.