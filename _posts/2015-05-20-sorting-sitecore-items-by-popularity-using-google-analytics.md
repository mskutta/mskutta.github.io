---
layout: post
title: "Sorting Sitecore Items by Popularity using Google Analytics"
date:   2015-05-20 00:00:00 -0500
categories: sitecore
tags: sitecore google analytics sorting
author: Mike Skutta
target: https://community.sitecore.net/technical_blogs/b/mike_skutta/posts/sorting-sitecore-items-by-popularity-using-google-analytics
excerpt: Not too long ago (prior to Sitecore 7.5) we had a requirement to add the ability to sort Sitecore items by Most Viewed and Most Shared. We got to thinking about what was the best way to implement something like this. In the past we may have created a custom database to track this information. Upon each page view or share, we could have written a row to the database. This data would then be available for use in sorting. This approach has costs involved with needing to maintain an extra database and an API. Could we take a simpler approach? We were already using Google Analytics to track page views and social interactions. Google provides APIs for its products, so we should be able to retrieve the data we need. We decided to go down the route of using Google Analytics to provide us with the data required for sorting by Mode Viewed and Most Shared.
---