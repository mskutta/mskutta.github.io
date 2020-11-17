---
layout: post
title: "Sitecore How-To: Setup Coveo in Sitecore Locally"
date:   2020-11-12 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Erik Carron and Mike Skutta
excerpt: This how-to will describe how to install/setup Coveo into a local Sitecore instance. This does not describe how to setup a local Coveo server. This process will be required when setting up a local Sitecore site that uses Coveo.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

This how-to will describe how to install/setup Coveo into a local Sitecore instance. This does not describe how to setup a local Coveo server. This process will be required when setting up a local Sitecore site that uses Coveo.

> This was the processed used to install Coveo 92 4.1.734.71 on to a Sitecore 9.2.0 site. It is possible that this process varies by Coveo/Sitecore version.

> It is assumed that the Sitecore databases already contain the correct Coveo items (i.e. the databases are from another Sitecore instance that had Coveo installed).

> It might be helpful to verify that you have access to a Coveo Cloud organization before attempting this: https://platform.cloud.coveo.com/. You will log in via your account. You can create a trial organization for 30 days if you do not have access to other organizations.

## Step-by-step guide

1. Delete any existing Coveo configuration files. They are typically in `\App_Config\Include\Coveo\` and `\App_Config\Modules\Coveo\`.
1. Download the `Coveo for Sitecore` Sitecore package. Make sure it is the correct version.
1. Install the package via Sitecore’s Installation Wizard.
    1. If prompted about overwriting dlls or configuration files, click `Yes to all`.
    1. When prompted about existing items in the database, select `Skip` and click `Apply to all`. (The databases should already contain the correct items, see note above) 
    1. When prompted about role `sitecore\Coveo Admin`, click `Continue Always`.
    1. When prompted to choose an organization, name it and choose the Cloud platform option. The instillation wizard will open a tab where you should login using Office 365 credentials. 
    1. At the end of the install process, the `Coveo for Sitecore Configuration` wizard will show. You need to complete this wizard to have Coveo fully configured. Failing to do so will require manual configuration as I do not know how to run this wizard outside of the package install process.
1. Finish the installation by restarting the server and client.
1. Depending on what work needs to be done, you might need to re-index Coveo related items using the indexing manager as well. (Warning: This might take several hours).
1. Build your project to deploy client specific Coveo configuration files.

If successfully installed, you should see the Coveo indexes in the Index Manager.

### Configuration Files

There are two locations where Coveo puts its configuration files:

1. `\App_Config\Include\Coveo`
    1. Contains the configuration files specific to your application. Some of these may be in source control.
    1. Without completing the configuration wizard, these files end in `.example`
    1. The following config files will be created/renamed once the configuration wizard is completed:
        1. `Coveo.CloudPlatformClient.Custom.config`
        1. `Coveo.SearchProvider.Custom.config`
        1. `Coveo.SearchProvider.Rest.Custom.config`
1. `\App_Config\Modules\Coveo`
    1. Contains the core Coveo configuration files. These should not be touched and should not be in source control.
    1. There are 4 config files that end in `.example` that Coveo will rename/enable once the configuration wizard is completed:
        1. `Coveo.SearchProvider.config`
        1. `Coveo.SearchProvider.Rest.config`
        1. `Coveo.UI.Components.ExperienceEditor.config`
        1. `Coveo.UI.Controls.config`

> If you do not complete the `Coveo for Sitecore Configuration` wizard at the end of the install, you will need to manually enable and edit these configuration files. The other option is to re-install the Coveo for Sitecore package and go through the configuration wizard at the end.