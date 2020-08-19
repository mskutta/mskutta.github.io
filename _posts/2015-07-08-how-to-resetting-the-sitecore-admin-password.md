---
layout: post
title: "How-To: Resetting the Sitecore Admin Password"
date:   2015-07-08 00:00:00 -0500
categories: sitecore
tags: sitecore password admin
author: Mike Skutta
excerpt: In web development, it is a best practice to reset the production password of the database whenever you are pulling it down for developers to use. When doing this in Sitecore, there is a simple procedure for resetting the Admin password back to the default password of “b.” Here’s how...
---

* content
{:toc}

## Overview

In web development, it is a best practice to reset the production password of the database whenever you are pulling it down for developers to use. When doing this in Sitecore, there is a simple procedure for resetting the Admin password back to the default password of "b" Here's how:

## Approach

The following SQL statement can be used to reset the "sitecore\admin" password to "b":</p>

``` sql
UPDATE [aspnet_Membership] SET Password='qOvF8m8F2IcWMvfOBjJYHmfLABc='   
WHERE UserId IN (SELECT UserId FROM [aspnet_Users] WHERE UserName = 'sitecore\Admin')
```

This approach has been confirmed to work on Sitecore 6.6, 7.2, and 7.5.
