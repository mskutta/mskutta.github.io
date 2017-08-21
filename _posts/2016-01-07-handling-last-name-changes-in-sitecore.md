---
layout: post
title: "Handling Last Name Changes in Sitecore"
date:   2016-01-07 00:00:00 -0500
categories: sitecore
tags: sitecore
author: Mike Skutta
image:
    url: /images/handling-last-name-changes-in-sitecore/content-tree-redirect.png
    height: 473
    width: 1024
excerpt: I wanted to share with you an approach for handling Last Name changes for items that represent people in Sitecore. The problem you usually face when changing a person’s name is what to do with their record after you’ve changed the name. An example of this is a maiden name change from Jane Smith to Jane Doe. Typically, the item name represents the name of the person. Jane Smith could have an item name of “Smith Jane.” The item names are used as part of the URL. To get to Jane Smith’s information on the public site, the URL may look like http://www.site.com/people/s/smith_jane.
---

* content
{:toc}

## Overview

I wanted to share with you an approach for handling Last Name changes for items that represent people in Sitecore. The problem you usually face when changing a person’s name is what to do with their record after you’ve changed the name. An example of this is a maiden name change from Jane Smith to Jane Doe. Typically, the item name represents the name of the person. Jane Smith could have an item name of “Smith Jane”. The item names are used as part of the URL. To get to Jane Smith’s information on the public site, the URL may look like: [http://www.site.com/people/s/smith_jane](http://www.site.com/people/s/smith_jane).

![Content Tree](/images/handling-last-name-changes-in-sitecore/content-tree.png)

There are a few potential problems with a name change:
* The item name / URL is now different than the old name.
* Changing the item name will 404 the old name.
* The item may not be in the logical last-name folder if the people reside in folders categorized by the previous last name.

## Solution

1. The following solution has worked well to address the above concerns.
Create a redirection data template.
    1. This template is meant to support the old name when items are renamed/moved.
    1. This data template has a pointer to the renamed/moved item.
    1. Items based on this template will respond to requests and redirect the user to the pointer reference. You would want to perform a 301 redirect if the change is permanent.
    1. Use an icon that represents redirection. This serves as a visual indicator to know that the person they are looking for had a name change: ![Icon](/images/handling-last-name-changes-in-sitecore/icon.png)
1. Rename and move the existing Person item to the appropriate last-name folder.
Create a new item based on the redirection data template in the old location using the old name.
1. Update the pointer to point at the renamed/moved item.

Any incoming request to the old URL will be handled by the redirection data template. The redirection data template will redirect the user to the renamed item.

![Content Tree Redirect](/images/handling-last-name-changes-in-sitecore/content-tree-redirect.png)

## Conclusion

The benefit of using this approach is to have a visual indicator in the old location of the item within the content tree that points to the new location. It is now clear to content administrators that there was a name change.