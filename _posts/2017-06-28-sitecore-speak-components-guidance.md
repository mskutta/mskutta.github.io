---
layout: post
title: "Sitecore SPEAK Components Guidance"
date:   2017-06-28 00:00:00 -0500
categories: sitecore
tags: sitecore speak
author: Mike Skutta
image:
    url: /images/sitecore-speak-components-guidance/dashboard.png
    height: 1024
    width: 816
---

* content
{:toc}

## Overview

I was recently working on a project building a custom dashboard in Sitecore. We decided to build the dashboard using SPEAK v2. The primary reason for this decision was we wanted a consistent user experience that followed the same look and feel as the rest of the Sitecore applications.  I received the designs of the new dashboard from our design team.  I quickly noticed that the designs were not completely in line with the components that are typically seen in SPEAK UIs.  I asked the designers if they were familiar with the **SPEAK Components Guidance**; they were not aware of it.  I realized that the **SPEAK Components Guidance** may not be well known.




## Components Guidance

The **SPEAK Components Guidance** is a Sitecore Application that demonstrates the use of the SPEAK components that are available out of the box with Sitecore.  This application is included in the base install.  You can get to the application following the link:

*http://[domain]/sitecore/client/Business%20Component%20Library/version%202/Content/Guidance/Dashboard*

![SPEAK Components Guidance Dashboard](/images/sitecore-speak-components-guidance/dashboard.png)

The navigation menu on the left provides examples of each of the components available.  Clicking on a component displays a demo and a reference.

![SPEAK Components Guidance Button](/images/sitecore-speak-components-guidance/button.png)

## Conclusion

When designing an application in Sitecore based on SPEAK, it is helpful to use the SPEAK Components Guidance as a reference.  Using the reference, we can decide if out of the box components can be used or if a custom component is required.

Bringing this to the attention of the designer allowed them to update the designs to primarily use the standard SPEAK components and only a handful of custom ones.

