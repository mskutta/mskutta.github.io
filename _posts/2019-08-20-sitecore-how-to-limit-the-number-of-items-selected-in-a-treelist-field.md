---
layout: post
title: "Sitecore How-To: Limit the Number of Items Selected in a Treelist Field"
date:   2019-08-20 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Kyle Mattimore and Mike Skutta
excerpt: How to limit the number of items selected in a treelist field or create a pseudo-single-select field that restricts what templates can be selected. 
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

How to limit the number of items selected in a treelist field or create a pseudo-single-select field that restricts what templates can be selected.

Currently Sitecore does not support restrictions like IncludeTemplatesForSelection and IncludeTemplatesForDisplay on any single select fields, so you can instead use a treelist that uses validation to restrict the number of items selected. You may also have other reasons to limit a picker to X amount of items

**Regular expression validation**

Replace the number 1 with any number to allow more than 1 item. (Note the pipe character is escaped)
```
^({[^}]+}\|?){0,1}$
```

## Step-by-step guide

1. Create a TreeList field that works for selecting multiple of the kind of items you want to pick.

    ```
    DataSource=/sitecore/content/Sites/Main/Home/Insights/Media Entities&IncludeTemplatesForSelection=MediaEntity&IncludeTemplatesForDisplay=MediaEntity,MediaEntityFolder
    ```
1. Add the regular expression in the validation field on the field item
    ![Validation](/images/sitecore-how-to-limit-the-number-of-items-selected-in-a-treelist-field/validation.png "Validation")
1. Try to violate your rule and make sure your warning pops up
