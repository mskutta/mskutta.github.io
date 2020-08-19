---
layout: post
title: "A Guide to Self-Hosting in Sitecore"
date:   2016-11-18 00:00:00 -0500
categories: sitecore
tags: sitecore marketing analytics
author: Mike Skutta, Rick Tham
excerpt: One North is a Sitecore Platinum Implementation Partner and Hosting Partner that has extensive experience hosting Sitecore on Microsoft Azure. We offer managed hosting services that benefit from the power, flexibility, performance and reliability of hosting Sitecore on the Microsoft Azure Cloud. While most of our clients choose our managed hosting offering on Azure, some may want full control and host Sitecore themselves. There are numerous ways that you can host and deploy Sitecore. Here are some options and trade-offs.
---

* content
{:toc}

## Overview

One North is a Sitecore Platinum Implementation Partner and Hosting Partner that has extensive experience hosting Sitecore on Microsoft Azure. We offer managed hosting services that benefit from the power, flexibility, performance and reliability of hosting Sitecore on the Microsoft Azure Cloud. While most of our clients choose our managed hosting offering on Azure, some may want full control and host Sitecore themselves.

There are numerous ways that you can host and deploy Sitecore. Here are some options and trade-offs.

## Basic Automated Deployment and Hosting Environment

This is a sample of a basic automated deployment and hosting environment.


1. **Version Control:** All code is stored in a version control system. Typically, the version control system would be GIT or TFS (Team Foundation Server). Environment branches would be used for production and QA environments. Development branches would be used for ongoing work.
1. **Build Server:** A build server would be utilized to build, test and deploy code updates to the various environments. TeamCity would typically be utilized here.
1. **Packaging and Deployment Tool:** Team Development for Sitecore (TDS), a tool developed to package and deploy Sitecore Items and code, would be used as part of the build process. This tool supports automated publishing of key items.
1. **QA Environment:** A QA environment would be utilized to test changes before the changes are taken live. In a simple setup, a single QA server would host both Content Management and Content delivery.
1. **Staging Environment:** A pre-production environment for final testing prior to deploying to production. This environment resembles production as close as possible. We recommend a staging environment. Some of our self-hosted clients have chosen not to include a staging environment.
1. **Production Environment:** A production environment would be set up based on requirements. The most basic setup would include a single Content Management server and two load balanced content delivery servers.

Here is a diagram of the above:

![Basic Automated Deployment Hosting Environment](/images/a-guide-to-self-hosting-in-sitecore/sitecorehosting_1.png)

## Advanced Automated Deployment and Hosting Environment

This is a sample of an advanced automated deployment and hosting environment.

1. **Version Control:** All code is stored in a version control system. Typically, the version control system would be GIT or TFS (Team Foundation Server). Environment branches would be used for production, staged and QA environments. Development branches would be used for ongoing work.
1. **Build Server:** A build server would be utilized to build, test and package code updates to the various environments. TeamCity would typically be utilized here.
1. **Packaging Tool:** Team Development for Sitecore (TDS), a tool developed to package Sitecore Items and code, would be used as part of the build process.
1. **Deployment Server:** A deployment tool such as Octopus Deploy would be used to deploy the packages to various environments. Benefits of using a deployment server include rolling deployments (updating the environment so no users are impacted) and rollback (in case something goes wrong).
1. **QA Environment:** A QA environment would be utilized to test changes before the changes are taken to staged. In a simple setup, a single QA server would host both Content Management and Content delivery.
1. **Staging Environment:** A pre-production environment for final testing prior to deploying to production. This environment resembles production as close as possible.
1. **Production Environment:** A production environment would be setup based on requirements. The setup would include multiple geo-distributed content delivery servers in the cloud. A single content management server would publish to the content delivery servers.

Here is a diagram of the above:

![Advanced Automated Deployment Hosting Enviornment Sitecore](/images/a-guide-to-self-hosting-in-sitecore/sitecorehosting_2.png)

This example is more difficult to setup and maintain. There is more control over the deployments as the deployment server can do rolling deployments and rollbacks. Deploying to the cloud allows for geo-distribution of the content delivery servers as well as the ability to scale on demand.
