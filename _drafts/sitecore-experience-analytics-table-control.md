---
layout: post
title: "Sitecore Experience Analytics Table Control"
date:   2017-07-17 00:00:00 -0500
categories: sitecore
tags: sitecore speak experience-analytics
author: Mike Skutta
---

* content
{:toc}

## Overview

Sitecore Experience Analytics reporting supports a list control that renders tabular data.  The data includes the **Key** along with *Visits, Value per visit, Average duration, Bounce Rate, Conversion Rate, and Page views per visit*.  Only standard analytics data can be displayed in the out of the box list control.  The columns are fixed and cannot be configured.




We had a requirement to customize the columns in the report.  We also had the requirement to display non-standard analytics data.  I did a bunch of research and did not find very many examples of how to achieve this.  I did find the following article: [Custom Dashboard Reports](https://community.sitecore.net/technical_blogs/b/integration_solution_team_blog/posts/custom-dashboard-reports).  It explains how to display aggregated data in a custom list. This is exactly what we needed.  The article was very helpful, but did not provide a working example.  I ended up needing to reflect into the [Sitecore Media Framework 2.1](https://dev.sitecore.net/Downloads/Sitecore_Media_Framework/21/Sitecore_Media_Framework_21.aspx) to find the missing pieces.

In an effort to help others, I am posting the complete and fully working code that I used to get this functioning properly.  I modified the code from the original [post](https://community.sitecore.net/technical_blogs/b/integration_solution_team_blog/posts/custom-dashboard-reports) to support using configurable types / methods instead of using SQL to provide the data.  This comes in handy if you want to display results from the **sitecore_analytics_index** Lucene index.

You can follow the original blog [post](https://community.sitecore.net/technical_blogs/b/integration_solution_team_blog/posts/custom-dashboard-reports) for an explanation of how this was built.  The *Sitecore Media Framework 2.1* that this example was based on is located here: [Sitecore Media Framework 2.1](https://dev.sitecore.net/Downloads/Sitecore_Media_Framework/21/Sitecore_Media_Framework_21.aspx).

![Basic Sample](/images/sitecore-experience-analytics-table-control/basic-sample.png)

I named the custom control **Experience Analytics Table Control** because the control returns tabular data with user-defined columns. A control by this name also does not exist yet.

### Features

The **Experience Analytics Table Control** supports rendering custom data within the Sitecore Experience Analytics dashboard.  The data is displayed in tabular form.  The columns in the table are customizable.  The look and feel matches the existing List Control.  The data is provided via configurable types / methods.  The methods provide the data that is displayed in the table.

## Source Code / Module

The source code for this control has been posted to: [https://github.com/onenorth/sitecore-experience-analytics-table-control](https://github.com/onenorth/sitecore-experience-analytics-table-control).  Please follow the instructions there for further information.  This control has also been released as a Sitecore Module: [https://marketplace.sitecore.net/Modules/E/Experience_Analytics_Table_Control.aspx?sc_lang=en](https://marketplace.sitecore.net/Modules/E/Experience_Analytics_Table_Control.aspx?sc_lang=en).  The **.update** package to install is located under releases here: [https://github.com/onenorth/sitecore-experience-analytics-table-control/releases](https://github.com/onenorth/sitecore-experience-analytics-table-control/releases).

