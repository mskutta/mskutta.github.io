---
layout: post
title: "Sitecore Deployment Process - Part 1"
date:   2017-07-18 00:00:00 -0500
categories: sitecore
tags: sitecore octopus-deploy team-city azure
author: Mike Skutta
---

* content
{:toc}

## Overview

One North, the agency I work for, is a [Sitecore Platinum Implementation Partner and Hosting Partner](https://www.onenorth.com/blog/post/one-north-interactive-named-sitecore-platinum-implementation-partner) that has extensive experience hosting Sitecore on Microsoft Azure.  We offer managed hosting services that benefit from the power, flexibility, performance and reliability of hosting Sitecore on the Microsoft Azure Cloud.

At the time of this writing, we host over 90 Sitecore sites in Azure. On average, we do over 30 Sitecore deployments a month. In order to support this volume and our continued growth, we needed a robust deployment strategy.




I wanted to share our approach of Deploying Sitecore to Azure.  This will be a multi-part series to cover the details.

## Software

First, lets talk about the software and tools that are part of our deployment process.

### Team City

[Team City](https://www.jetbrains.com/teamcity/) is a distributed build management and continuous integration server.  It has the ability to pull code from version control, compile the code, run tests against the code, and make the results available for further consumption. We chose to use Team City because it can compile and package our code the same way each and every time in a consistent manner. It eliminates developer error setting up projects manually for production builds.  It also supports more than one build agent.  This helps when more than one project needs to be built at the same time. 

There are several options for build servers out there.  Team City is one of the more popular ones.

### Octopus Deploy

[Octopus Deploy](https://octopus.com/) is an automated deployment tool for .NET.  it takes the results from a build server, such as Team City, and automates the deployment to one or more environments.  Environments typically consist of test, staging, and production.  The environments can be in the cloud or on-premises.



