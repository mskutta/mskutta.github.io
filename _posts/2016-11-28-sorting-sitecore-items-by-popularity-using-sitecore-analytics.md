---
layout: post
title: "Sorting Sitecore Items by Popularity Using Sitecore Analytics"
date:   2016-11-28 00:00:00 -0500
categories: sitecore
tags: sitecore hosting
author: Mike Skutta
source: https://www.onenorth.com/blog/post/sorting-sitecore-items-by-popularity-using-sitecore-analytics
excerpt: A while back, I created a blog post that explained how to Sort Sitecore Items by Popularity using Google Analytics. In this post, I will explain how to achieve the same goal using xDB. As a quick recap, clients may have a requirement to sort Sitecore items by Most Viewed. In many instances, Sitecore Analytics and xDB are already being used to track analytics information.  We should be able to retrieve the data we need from xDB as well as the reporting database. There may be alternate ways to retrieve the data from xDB and the reporting database.  I am going to take the route that includes adding a Page Event and a custom Fact Table, demonstrating how they function.
---
