---
layout: post
title: "How to Run Quick Sitecore Reports Without Any Coding"
date:   2017-08-09 00:00:00 -0500
categories: sitecore
tags: sitecore report xpath
author: Mike Skutta and Alex Pershteyn
image:
    url: /images/how-to-run-quick-sitecore-reports-without-any-coding/xpath-builder.png
    height: 784
    width: 1040
excerpt: During an exercise with our Sitecore configuration we notice a bug with the patch instead action that is described here  http://sitecore.stackexchange.com/questions/2049/patchinstead-removes-an-element-with-no-attributes
---

* content
{:toc}

## Overview

Sometimes you need to run a quick report from Sitecore. For example - find all news published between 7/20/2017 and 7/24/2017. The easiest way to accomplish it is to use built-in XPath builder tool.

![XPath Builder](/images/how-to-run-quick-sitecore-reports-without-any-coding/xpath-builder.png)

## Step-by-Step Guide

1. Login as admin and navigate to [http://[YOUR-SITECORE-INSTANCE]/sitecore/shell/default.aspx?xmlcontrol=IDE.XPath.Builder](http://[YOUR-SITECORE-INSTANCE]/sitecore/shell/default.aspx?xmlcontrol=IDE.XPath.Builder)
1. Use Sitecore query to get content that you are looking for. 
    1. [http://sitecoreworld.blogspot.com/2014/09/querying-items-from-sitecore.html](http://sitecoreworld.blogspot.com/2014/09/querying-items-from-sitecore.html)
    1. [https://sdn.sitecore.net/upload/sdn5/developer/using%20sitecore%20fast%20query/using%20sitecore%20fast%20query001.pdf](https://sdn.sitecore.net/upload/sdn5/developer/using%20sitecore%20fast%20query/using%20sitecore%20fast%20query001.pdf)

> Note: Never run this on the live site before testing locally or on qa first. Certain queries may take a while to run and could cause significant performance issues.

## Samples

Here are a few queries I've used:

1. Get all news published between 08/07/2015 and 04/15/2016: 
    ``` text
    /sitecore/content/home/news//*[@__Created >= '20150807' and @__Created <= '20160415' and @@templateid = '{########-####-####-####-############}']
    ```
1. Find all people created after 11/01/2016:
    ``` text
    /sitecore/content/home/people//*[@@templateid = '{########-####-####-####-############}' AND @__Created>='20161101']
    ```
1. Find all people with first name ‘John’: 
    ``` text
    /sitecore/content/home/people//*[@@templateid = '{########-####-####-####-############}' AND @FirstName='John']
    ```
1. Find all fields with Field Fallback disabled:
    ``` text
    /sitecore/templates/User Defined//*[@@templatename='Template field' and @Enable Shared Language Fallback!='1']
    ```
1. Find all people whose last name contains 'Co':
    ``` text
    /sitecore/content/home/people//*[@@templateid = '{########-####-####-####-############}' AND contains(@LastName, "Co")]
    ```