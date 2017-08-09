---
layout: post
title: "Data Integration using the Sitecore Publish Pipeline"
date:   2015-06-11 00:00:00 -0500
categories: sitecore
tags: sitecore data integration
author: Mike Skutta
target: https://community.sitecore.net/technical_blogs/b/mike_skutta/posts/data-integration-using-the-sitecore-publish-pipeline
excerpt: I recently used the Sitecore Publish Pipeline to do a data integration with other systems in our network. While going through this process, it occurred to me that a how-to might be helpful to others trying to do the same. We had a requirement where we needed published content to be synced with 3rd party systems in near real-time. The 3rd party system essentially needed the content from the web database. The best way to sync the data in near real-time was to plug into the process that Sitecore used to populate the web database. This would ensure that we would have access to the data as it was being published. Sitecore uses the publish pipeline to migrate published items from the master database to the web database. This would be the perfect spot to plug in our logic.
---