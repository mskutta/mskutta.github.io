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

### Bitbucket

![Bitbucket](/images/sitecore-deployment-process-part-1/Bitbucket_400x400.jpg){:height="200px" width="200px"}

[Bitbucket](https://bitbucket.org/) is a service that hosts Git repositories.  It is similar to GitHub, but it is built for professional teams.  We like Bitbucket because it allows unlimited private repositories. We chose to use Git as our version control system because of its wide use, popularity, and that most developers have experience with it. Environment branches are used for production, staging, and qa environments. Development branches are used for ongoing work.

### Team City

![Team City](/images/sitecore-deployment-process-part-1/TeamCity_400x400.jpg){:height="200px" width="200px"}

[Team City](https://www.jetbrains.com/teamcity/) is a distributed build management and continuous integration server.  It has the ability to pull code from version control, compile the code, run tests against the code, and make the results available for further consumption. We chose to use Team City because it can compile and package our code the same way each and every time in a consistent manner. It eliminates developer error setting up projects manually for production builds.  It also supports more than one build agent.  This helps when more than one project needs to be built at the same time. 

There are several options for build servers out there.  Team City is one of the more popular ones.

### Octopus Deploy

![Octopus Deploy](/images/sitecore-deployment-process-part-1/OctopusDeploy_400x400.png){:height="200px" width="200px"}

[Octopus Deploy](https://octopus.com/) is an automated deployment tool for .NET. It takes the results from a build server, such as Team City, and automates the deployment to one or more environments. Environments typically consist of test, staging, and production.  The environments can be in the cloud or on-premises. The deployment process is completely customizable and scriptable via PowerShell.  Octopus Deploy also takes care of storing sensitive production settings.  Octopus can include these settings into the configuration as part of the deployment process.

### Azure

![Azure](/images/sitecore-deployment-process-part-1/Azure_400x400.png){:height="200px" width="200px"}

[Azure](https://azure.microsoft.com/) is an open, flexible, enterprise-grade cloud computing platform.  It supports PaaS (Platform as a Service) and IaaS (Infrastructure as a Service).  It has data centers Geographically distributed in many regions.  Azure supports scaling up and/or out as well as providing geo-redundancy.  We use virtual machines in Azure to host our Sitecore instances.  We are able to support hosting a Sitecore instance on one or more machines based on client requirements. 

## Process

The typical deployment process goes down the chain.  First, working code is committed and pushed to BitBucket.  Team City picks up the code changes, and pulls the code from BitBucket.  Team City then compiles the code.  After that, it makes sure the code passes various test cases.  Finally it packages the code up and sends the package over to Octopus Deploy.  Octopus Deploy holds onto the package until an administrator tells it to deploy the code to an environment.  Once told to do a deployment, Octopus Deploy handles configuring the Virtual Machines in the Azure environment as well as deploying the updated code.  Azure then hosts the Sitecore site.

TODO: GRAPHIC




