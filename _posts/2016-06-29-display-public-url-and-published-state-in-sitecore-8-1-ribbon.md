---
layout: post
title: "Display Public URL and Published State in Sitecore 8.1 Ribbon"
date:   2016-06-29 00:00:00 -0500
categories: sitecore
tags: sitecore ribbon url
author: Mike Skutta
image:
    url: /images/display-public-url-and-published-state-in-sitecore-8-1-ribbon/display-public-url-and-published-state.png
    height: 520
    width: 864
excerpt: We ran across a helpful blog post by Kevin Buckley explaining how to display the published state of an item in the Sitecore ribbon using a ribbon panel. This ribbon panel displays the version published by language if the item has been published. If the items is not published, it indicates that. We implemented this and have found it to be very useful for our content administrators.
---

* content
{:toc}

## Overview

We ran across a helpful blog post by Kevin Buckley explaining how to display the published state of an item in the Sitecore ribbon using a ribbon panel. [https://webcmd.wordpress.com/2011/08/31/sitecore-ribbon-that-displays-published-state-of-an-item/](https://webcmd.wordpress.com/2011/08/31/sitecore-ribbon-that-displays-published-state-of-an-item/). This ribbon panel displays the version published by language if the item has been published. If the items is not published, it indicates that. We implemented this and have found it to be very useful for our content administrators.

We expanded upon the code provided in the post. In addition to displaying the published state, we also added the public facing URL to the ribbon. This allows content administrators to see the published state as well as the public URL to the item. The URL not only benefits content items, but also Media Items. Once you upload a media item you can get the public URL to the Media Item, without having to include it on a page. We also updated the code to work with the new look and feel in Sitecore 8.1. 

![Sitecore Ribbon](/images/display-public-url-and-published-state-in-sitecore-8-1-ribbon/display-public-url-and-published-state.png)

## Implementation

Here is the updated code:

``` csharp
using System;
using System.Linq;
using System.Web.UI;
using Sitecore;
using Sitecore.Configuration;
using Sitecore.Data;
using Sitecore.Data.Items;
using Sitecore.Diagnostics;
using Sitecore.IO;
using Sitecore.Links;
using Sitecore.Publishing;
using Sitecore.Resources.Media;
using Sitecore.Shell.Framework.Commands;
using Sitecore.Shell.Web.UI.WebControls;
using Sitecore.Sites;
using Sitecore.Web.UI.WebControls.Ribbons;
using Website.Logic.Common.Extensions;

namespace Website.Logic.ContentEditor
{
    public class PublishInfo : RibbonPanel
    {
        /// <summary>
        /// Publish Info - see here for details:
        /// http://webcmd.wordpress.com/2011/08/31/sitecore-ribbon-that-displays-published-state-of-an-item/
        /// </summary>
        /// <param name="output"></param>
        /// <param name="ribbon"></param>
        /// <param name="button"></param>
        /// <param name="context"></param>
        public override void Render(HtmlTextWriter output, Ribbon ribbon, Item button, CommandContext context)
        {
            // Validate Arguments
            Assert.ArgumentNotNull(output, "output");
            Assert.ArgumentNotNull(ribbon, "ribbon");
            Assert.ArgumentNotNull(button, "button");
            Assert.ArgumentNotNull(context, "context");

            try
            {
                var htmlLinkOutput = string.Empty;
                var htmlPublishOutput = string.Empty;

                // Confirm item exists in the context
                var contextItem = context.Items.FirstOrDefault();
                if (contextItem != null)
                {

                    var url = string.Empty;

                    if (contextItem.Paths.IsContentItem)
                    {
                        // Find the best matching site
                        var contextItemPath = contextItem.Paths.Path.ToLower();
                        var site = SiteContextFactory.Sites
                            .FirstOrDefault(s => s.VirtualFolder == "/" && s.RootPath != "" && contextItemPath.StartsWith(FileUtil.MakePath(s.RootPath, s.StartItem).ToLower()));
                          
                        if (site != null)
                        {
                            var urlOptions = LinkManager.GetDefaultUrlOptions();
                            urlOptions.Site = new SiteContext(site);
                            url = LinkManager.GetItemUrl(contextItem, urlOptions);
                        }
                                
                    }
                    else if (contextItem.Paths.IsMediaItem && contextItem.TemplateID != TemplateIDs.MediaFolder)
                    {
                        using (new SiteContextSwitcher(Factory.GetSite("website")))
                            url = MediaManager.GetMediaUrl(contextItem);
                    }

                    // Create the HTML output to render.
                    if (!string.IsNullOrEmpty(url))
                    {
                        htmlLinkOutput = string.Format("<div style='padding:3px 3px 5px 7px;display: inline-block;border:1px solid;'>" +
                                                            "<div style='padding:3px 0px 5px 0px;'>Relative Link to this page</div>" +
                                                            "<div style='font-weight:bold'>{0}</div>" +
                                                        "</div>&nbsp;&nbsp;", url);
                    }
                    
                    // Determine if the item is published

                    // Obtain reference to the master database
                    var masterDb = Database.GetDatabase("master");

                    // Find all of the publishing targets and determine if the item has been published to those targets
                    var publishingTargets = PublishManager.GetPublishingTargets(masterDb);
                    foreach (var publishingTarget in publishingTargets)
                    {
                        var targetDatabaseName = publishingTarget.GetValueOrDefault<string>("Target database");
                        var isPreviewTarget = publishingTarget.GetValueOrDefault<bool>("Preview publishing target");
                        if (!string.IsNullOrEmpty(targetDatabaseName) && !isPreviewTarget)
                        {
                            var targetDatabase = Database.GetDatabase(targetDatabaseName);
                            if (targetDatabase != null)
                            {
                                //SelectSingleItem does a direct request to the database for the item
                                var item = targetDatabase.SelectSingleItem(contextItem.ID.ToString());
                                if (item != null)
                                {
                                    foreach (var language in item.Languages)
                                    {
                                        var languageVersion = item.Versions.GetLatestVersion(language);
                                        if (languageVersion != null && languageVersion.Versions.Count > 0)
                                        {
                                            htmlPublishOutput += string.Format("<div>{0} - {1}</div>", languageVersion.Version.Number, languageVersion.Language.CultureInfo.DisplayName);
                                        }
                                    }
                                }
                                // Exit after the first publishing target
                                break;
                            }
                        }
                    }
                    if (string.IsNullOrEmpty(htmlPublishOutput))
                        htmlPublishOutput = string.Format("<div>{0}</div>", "No");

                    htmlPublishOutput = string.Format("<div style='padding:3px 3px 5px 7px;display: inline-block;border:1px solid;'>" +
                                                        "<div style='padding:3px 0px 5px 0px;font-weight:bold'>Published to Web</div>" +
                                                            "{0}" +
                                                        "</div>", htmlPublishOutput);
                }

                var htmlOutput =
                                string.Format(
                                    "<div class='scRibbonToolbarText' style='padding:0 10px 7px 5px;height:auto;border:none !important;float:none;display:inline-block;'>" +
                                       "{0}" +
                                       "{1}" +
                                    "</div>",
                                    htmlLinkOutput, htmlPublishOutput);

                output.Write(htmlOutput);
            }
            catch (Exception ex)
            {
                Log.Error("Exception in custom ItemUrlInfo Ribbon: " + ex.Message, this);
            }
        }
    }
}
```

To configure the ribbon panel, perform the following steps:

1. Switch to the Core database
1. Navigate to: /sitecore/content/Applications/Content Editor/Ribbons/Chunks/Publish
1. Insert a new item based on the template: /sitecore/templates/System/Ribbon/Panel
1. Set the Type to the type of the class.

## Special Thanks

I would like to thank Kevin Buckley, the author of [https://webcmd.wordpress.com/2011/08/31/sitecore-ribbon-that-displays-published-state-of-an-item/](https://webcmd.wordpress.com/2011/08/31/sitecore-ribbon-that-displays-published-state-of-an-item/) for the idea and code.