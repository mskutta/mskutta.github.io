---
layout: post
title: "Sitecore: Workflow Full History"
date:   2015-08-26 00:00:00 -0500
categories: sitecore
tags: sitecore history
author: Mike Skutta
image:
    url: /images/sitecore-workflow-full-history/workflow-full-history-dialog.png
    height: 350
    width: 352
excerpt: Our clients were asking for a way to view Workflow History for all versions of an item at the same time. Out of the box, Sitecore provides a way to view the Workflow History for the current version of an item. We decided to create an additional “Full History” button in the Workflow group that allowed our clients to view the full history. The Full History button only appears for items that are participating in workflow.
---

* content
{:toc}

## Overview

Our clients were asking for a way to view Workflow History for all versions of an item at the same time. Out of the box, Sitecore provides a way to view the Workflow History for the current version of an item. We decided to create an additional “Full History” button in the Workflow group that allowed our clients to view the full history. The Full History button only appears for items that are participating in workflow.

![Sitecore Ribbon](/images/sitecore-workflow-full-history/workflow-full-history-button.png)

Clicking the Full History button displays a pop-up with history for each version of the item.

![Sitecore Ribbon](/images/sitecore-workflow-full-history/workflow-full-history-dialog.png)

The first thing we need to do is implement the logic for the button that will appear in the ribbon. To do so, we first need to create a class that inherits from **Sitecore.Shell.Framework.Commands.Command**. We then need to override a couple methods.

The first method that we need to override is the **QueryState** method. The result of this method call tells the framework whether the “Full History” button should show or not.

``` csharp
public override CommandState QueryState(CommandContext context)
{
    if (context.Items.Length != 1)
        return CommandState.Hidden;

    var item = context.Items[0];

    var workflowProvider = item.Database.WorkflowProvider;
    if (item == null || workflowProvider == null || workflowProvider.GetWorkflow(item) == null)
        return CommandState.Hidden;

    return CommandState.Enabled;
}
```

There are a few things to check: Does an item exist? Is there a workflow provider? Is there a workflow for the current item? **CommandState.Enabled** or **CommandState.Hidden** is returned indicating if the button should show or hide.

The next method we need to override is the Execute method. This method is called when the button is clicked. We want to show a popup as a result of the click. To do this we need to start a UI pipeline. This is done with the **Sitecore.Context.ClientPage.Start** method. The method takes in a method name and the parameters to pass to the method. The method name indicates what method to call as part of the pipeline. In our Execute method, all we are doing is preparing the parameters and starting the pipeline.

``` csharp
public override void Execute(CommandContext context)
{
    var item = context.Items[0];

    var parameters = new System.Collections.Specialized.NameValueCollection();
    parameters["database"] = item.Database.Name;
    parameters["itemid"] = item.ID.ToString();
    parameters["language"] = item.Language.Name;
    Sitecore.Context.ClientPage.Start(this, "Run", parameters);
}
```

The passed parameters are required in the Run method.

Lastly, we need to implement the logic in the Run method. The logic retrieves all versions of the current item. For each version, it calls the Workflow Provider to get the history. Embedded HTML templates are used to format the results. Lastly, **Sitecore.Context.ClientPage.ClientResponse.ShowPopup** is called to display the results.

``` csharp
protected void Run(ClientPipelineArgs args)
{
    if (args.IsPostBack)
        return;

    var database = Database.GetDatabase(args.Parameters["database"]);
    var itemId = ID.Parse(args.Parameters["itemid"]);
    var language = Sitecore.Globalization.Language.Parse(args.Parameters["language"]);

    var workflowProvider = (database.WorkflowProvider as WorkflowProvider);
    if (workflowProvider == null || workflowProvider.HistoryStore == null)
        return;

    var item = database.GetItem(itemId, language);

    var assembly = Assembly.GetExecutingAssembly();
    string itemHtml;
    using (var stream = assembly.GetManifestResourceStream("OneNorth.WorkflowFullHistory.Shell.Framework.Commands.WorkflowFullHistoryItem.html"))
    using (var reader = new StreamReader(stream))
        itemHtml = reader.ReadToEnd();

    var html = new StringBuilder();
    foreach (var version in item.Versions.GetVersions())
    {
        var workflowEvents = workflowProvider.HistoryStore.GetHistory(version);

        foreach (var workFlowEvent in workflowEvents)
        {
            var oldStateItem = (ID.IsID(workFlowEvent.OldState))
                                   ? database.GetItem(ID.Parse(workFlowEvent.OldState))
                                   : null;
            var oldState = (oldStateItem != null) ? oldStateItem.DisplayName : "?";

            var newStateItem = (ID.IsID(workFlowEvent.NewState))
                                   ? database.GetItem(ID.Parse(workFlowEvent.NewState))
                                   : null;
            var newState = (newStateItem != null) ? newStateItem.DisplayName : "?";
            var iconUrl = (newStateItem != null) ? Sitecore.Resources.Images.GetThemedImageSource(newStateItem.Appearance.Icon, ImageDimension.id24x24) : "";

            html.AppendFormat(itemHtml, 
                iconUrl,
                workFlowEvent.User, 
                workFlowEvent.Date.ToString("D"),
                version.Version.Number, 
                oldState, 
                newState, 
                HttpUtility.HtmlEncode(workFlowEvent.Text));
        }
    }

    string popupHtml;
    using (var stream = assembly.GetManifestResourceStream("OneNorth.WorkflowFullHistory.Shell.Framework.Commands.WorkflowFullHistoryPopup.html"))
    using (var reader = new StreamReader(stream))
        popupHtml = reader.ReadToEnd();

    Sitecore.Context.ClientPage.ClientResponse.ShowPopup(Guid.NewGuid().ToString(), "below", string.Format(popupHtml, html));
}
```

The complete code, including the html templates, are located here: 
[https://github.com/onenorth/workflow-full-history/tree/master/src/Website/Shell/Framework/Commands](https://github.com/onenorth/workflow-full-history/tree/master/src/Website/Shell/Framework/Commands)

Now that we have the code, we need to register the command and create a button to execute the command. We can register the command in an App_Config/Include configuration file.

``` xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <commands>
      <command name="item:workflowfullhistory" type="OneNorth.WorkflowFullHistory.Shell.Framework.Commands.WorkflowFullHistory, OneNorth.WorkflowFullHistory" />
    </commands>
  </sitecore>
</configuration>
```

1. Switch to the “core” database.
1. Open the Content Editor.
1. Navigate to the following location: sitecore > content > Applications > Content Editor > Ribbons > Chunks > Workflow
1. Add the following child item: Template: /sitecore/templates/System/Ribbon/Small Button Name: Full History
1. Update the item with the following values:
    Header: Full History
    Icon: Applications/32x32/history2.png
    Click: item:workflowhistory

The button should now appear in the ribbon in the “Workflow” chunk. Remember the button only appears when the context item participates in workflow.

![Sitecore Ribbon](/images/sitecore-workflow-full-history/workflow-full-history-ribbon.png)

Test the button and try it out.

## Conclusion

As you can see it is very easy to add a button that shows the full workflow history for an item.  I hope you find this useful.  The complete source code is located at: [https://github.com/onenorth/workflow-full-history](https://github.com/onenorth/workflow-full-history).  The code has also been released as an update module for easy installation: [https://github.com/onenorth/workflow-full-history/releases](https://github.com/onenorth/workflow-full-history).  See the readme for instructions on installation.
