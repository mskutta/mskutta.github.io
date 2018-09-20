---
layout: post
title: "Sitecore How-To: Creating Custom Workflows"
date:   2018-09-20 00:00:00 -0500
categories: sitecore
tags: sitecore-how-to
author: Matt Weeks and Mike Skutta
excerpt: Creating a custom workflow in sitecore might be useful in a scenario where just publishing the item is not enough. In my case I needed to publish a sub item, then publish its parent entity, and finally publish a related item on the initial sub item.
---

* content
{:toc}

## Overview

This is an article in a series of Sitecore how-to articles. These articles are meant to be quick guides to accomplish various tasks within Sitecore. The how-to articles have proven to be very helpful internally at *One North Interactive* https://www.onenorth.com.  These articles assume working Sitecore knowledge. I just wanted to share the articles with the community. Hopefully you find them helpful.

## How-To

Creating a custom workflow in Sitecore might be useful in a scenario where just publishing the item is not enough. In my case, I needed to publish a sub item, then publish its parent entity, and finally publish a related item on the initial sub item.

## Step-by-step guide

1. Create a new workflow item in the 
    1. location /sitecore/system/Workflows/
    1. template /sitecore/templates/System/Workflow/Workflow
1. Under that item, add two workflow states.
    1. template /sitecore/templates/System/Workflow/State
    1. Draft State, make sure the final check box is not selected. ![Workflow Draft](/images/how-to-creating-custom-workflows-in-sitecore/Workflow Draft.PNG)
    1. Publish State, make sure the final check box is selected. (This is the final workflow state) ![Workflow Publish](/images/how-to-creating-custom-workflows-in-sitecore/Workflow Publish.PNG)
1. Using the workflow created in step 1, set an Initial state.
    1. ![Workflow](/images/how-to-creating-custom-workflows-in-sitecore/Workflow.PNG)
    1. The initial state should point to the Draft state created in step 2.1.
1. Under the Draft state item created in step 2.2, add a Workflow command.
    1. template /sitecore/templates/System/Workflow/Command
    1. Set the Next state field to the Publish state created in step 2.3. ![Workflow Draft Command](/images/how-to-creating-custom-workflows-in-sitecore/Workflow Draft Command.PNG)
1. Add the publish action to publish the initial item; this uses the publish action from the Sitecore kernel.
    1. the type should be "Sitecore.Workflows.Simple.PublishAction, Sitecore.Kernel"
1. After the publish action, add a custom action; in my case it is titled Publish Parent Entity.
    1. ![Workflow Custom Action](/images/how-to-creating-custom-workflows-in-sitecore/Workflow Custom Action.PNG)
    1. Notice the Type string is set to a class contained in the website project.
    1. The code below gets the item being published, it then finds its parent item and publishes it.

    ```c#
    using Sitecore.Data.Items;
    using Sitecore.Workflows.Simple;
    using Website.Logic.Common.Extensions;
    
    namespace Website.Logic.Common.Workflow
    {
        public class PublishParentEntityAction
        {
            public PublishParentEntityAction()
            {
            }
    
            public virtual void Process(WorkflowPipelineArgs args)
            {
                Item dataItem = args.DataItem;
    
                var parent = dataItem.Parent;
                
                if (parent != null)
                {
                    PublishUtility.PublishItem(parent);                   
                }
            }
        }
    }
    ```