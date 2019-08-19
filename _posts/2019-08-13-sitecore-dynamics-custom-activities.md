---
layout: post
title: "Sitecore: Dynamics Custom Activity Integration"
date:   2019-08-13 00:00:00 -0500
categories: sitecore
tags: sitecore dynamics xconnect data-exchange-framework
author: Mike Skutta
excerpt: Sitecore provides a connector to integrate Microsoft Dynamics with Sitecore.  The connector is called Sitecore Connect for Microsoft Dynamics 365 for Sales.  The out-of-the-box implementation provides a good set of standard mappings between Sitecore and Dynamics.  There are not many examples of customizing the integration.  Sitecore does provide an example of creating a custom facet for population from Dynamics. I had a requirement to bring additional events from Sitecore into Dynamics Activities. I could not find any good examples to follow. I reverse engineered what Sitecore already had for Email Activities in order to support additional events. I just wanted to share what I found in case others are looking to do the same.
---

* content
{:toc}

## Overview

Sitecore provides a connector to integrate Microsoft Dynamics with Sitecore.  The connector is called *Sitecore Connect for Microsoft Dynamics 365 for Sales* and can be downloaded [here](https://dev.sitecore.net/Downloads/Dynamics_CRM_Connect.aspx).  The out-of-the-box implementation provides a good set of standard mappings between Sitecore and Dynamics as documented [here](https://doc.sitecore.com/developers/dynamics-crm-connect/20/sitecore-connect-for-microsoft-dynamics-365-for-sales/en/microsoft-dynamics-365-for-sales-crm-connect-configuration-guide.html). There are not many examples of customizing the integration.  Sitecore does provide an [example](https://doc.sitecore.com/developers/dynamics-crm-connect/20/sitecore-connect-for-microsoft-dynamics-365-for-sales/en/walkthrough--creating-a-custom-contact-facet.html) of creating a custom facet for population from Dynamics.   I had a requirement to bring additional events from Sitecore into Dynamics Activities. I could not find any good examples to follow. I reverse engineered what Sitecore already had for [Email Activities](https://doc.sitecore.com/developers/dynamics-crm-connect/20/sitecore-connect-for-microsoft-dynamics-365-for-sales/en/email-activity.html) in order to support additional events. I just wanted to share what I found in case others are looking to do the same.

## Approach

*Sitecore Connect for Microsoft Dynamics 365 for Sales* is built on top of the *Sitecore Data Exchange Framework* or *DEF*.  The DEF uses the Sitecore content tree to define pipelines and configuration that are used to drive data integrations.  Code backs these pipelines and configuration. There is a convention to follow when creating a provider and tenant for the DEF. The Dynamics connector follows this convention.  We can walk into the content tree for the email activity to determine what needs to be done for a custom activity.  Below is a screen shot of the content tree for a Dynamics tenant.

![Dynamics Tenant](/images/sitecore-dynamics-custom-activity/dynamics-tenant.png "Dynamics Tenant")

## Implementation

The Email Activity is synced from Sitecore to Dynamics. First, let's start at the entry point for running the Pipeline Batches.  The entry point we are interested in is:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipeline Batches/xConnect Contacts to Dynamics Sync
```

This is the node in which you can *Run Pipeline Batch*.

![xConnect Contacts to Dynamics Sync](/images/sitecore-dynamics-custom-activity/xconnect-contacts-to-dynamics-sync.png "xConnect Contacts to Dynamics Sync")

From this node, the pipeline that is run is:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipelines/xConnect Contacts to Dynamics Sync Pipelines/Read Contacts from xConnect Pipeline
```

The pipeline steps are under the **Read Contacts from xConnect Pipeline** node.

![Read Contacts from xConnect Pipeline](/images/sitecore-dynamics-custom-activity/read-contacts-from-xconnect-pipeline.png "Read Contacts from xConnect Pipeline")

None of these pipeline steps have anything specific with Email Activities.  The next pipeline that is called is defined in **Iterate xConnect Contacts and Run Pipelines**.  The pipeline that is called is:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipelines/xConnect Contacts to Dynamics Sync Pipelines/Process Single Contact from xConnect Pipeline
```  

![Read Contacts from xConnect Pipeline](/images/sitecore-dynamics-custom-activity/process-single-contact-from-xconnect-pipeline.png "Read Contacts from xConnect Pipeline")

These pipeline steps are also not specific to Email Activities.  The next pipeline that is called is:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipelines/xConnect Contacts to Dynamics Sync Pipelines/Process Single Interaction from xConnect Pipeline
```

![Process Single Interaction from xConnect Pipeline](/images/sitecore-dynamics-custom-activity/process-single-interaction-from-xconnect-pipeline.png "Process Single Interaction from xConnect Pipeline")

We have finally found a reference to Emails.  This is where we can start following the existing structure and duplicate it for our custom event.  We will use the page viewed event for this example.

The **Iterate xConnect Interactions and Run Pipelines** item's Pipelines field indicates that "The selected pipelines are run for each member of the iterable data."  From this, and the fact the field is a multi-select, we can make the assumption that we can add another custom pipeline to that list.

### Create Interaction Pipeline

Following the existing pattern, we should be able to duplicate **Process Single Interaction from xConnect Pipeline** and create a modified custom version.  I duplicated **Process Single Interaction from xConnect Pipeline** and named the duplicate **Process Single Page View Interaction from xConnect Pipeline**.  I appended **- 1N** to each of the root items I created to better keep track of them.  I updated each of the child steps renaming **Email Sent** to **Page View**:

* Read Email Sent Events from Interaction > Read Page View Events from Interaction
* Iterate Email Sent Events and Run Pipelines > Iterate Page View Events and Run Pipelines

I then registered the new pipeline in the **Iterate xConnect Interactions and Run Pipelines** item.

![Process Single Page View Interaction from xConnect Pipeline](/images/sitecore-dynamics-custom-activity/process-single-page-view-interaction-from-xconnect-pipeline.png "Process Single Page View Interaction from xConnect Pipeline")

The following steps need to be customized to support **Page View Events**:

* Read Page View Events from Interaction
* Iterate Page View Events and Run Pipelines

The **Source Object Value Accessor** needs to be updated to support **Page View Events**.  The **Email Sent Events** version points to:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Accessors/Providers/xConnect/Email Sent Events from Interaction
```

![Read Email Sent Events from Interaction](/images/sitecore-dynamics-custom-activity/read-email-sent-events-from-interaction.png "Read Email Sent Events from Interaction")

This is a filter that only keeps Email Sent Events.  We need to create a new filter to only keep Page Viewed Events.

### Create PageViewEvent Filter

Duplicate the **Email Sent Events from Interaction** and name the copy **Page View Events from Interaction**.

![Email Sent Events from Interaction](/images/sitecore-dynamics-custom-activity/email-sent-events-from-interaction.png "Email Sent Events from Interaction")

Now we need to create a Filter specific to **Page View Events**.  The existing filter is for **Email Sent Events**.  Duplicate the filter:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Tenant Settings/Providers/xConnect/Filter Expressions/Event Filter Expressions/Email Sent Event Expression
```

Name the copy **Page View Event Expression**.  The **Event Type** in the copy needs to be updated to point to the PageView Event.  Through reflection, I was able to figure out the available types.  The type I needed for this is:

```
Sitecore.XConnect.Collection.Model.PageViewEvent, Sitecore.XConnect.Collection.Model
```

Update the Event Type filed with this type.

![Page View Event Expression](/images/sitecore-dynamics-custom-activity/page-view-event-expression.png "Page View Event Expression")

Now go back and update the **Filter** in:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Accessors/Providers/xConnect/Page View Events from Interaction - 1N
```

Set the value of the **Filter** to:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Tenant Settings/Providers/xConnect/Filter Expressions/Event Filter Expressions/Page View Event Expression - 1N
```

![Page View Events from Interaction](/images/sitecore-dynamics-custom-activity/page-view-events-from-interaction.png "Page View Events from Interaction")

Go back to:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipelines/xConnect Contacts to Dynamics Sync Pipelines/Process Single Page View Interaction from xConnect Pipeline - 1N/Read Page View Events from Interaction
```

Update the **Source Object Value Accessor** with:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Accessors/Providers/xConnect/Page View Events from Interaction - 1N
```

![Read Page View Events from Interaction](/images/sitecore-dynamics-custom-activity/read-page-view-events-from-interaction.png "Read Page View Events from Interaction")

### Create Single Event Pipeline

The **Iterate Page View Events and Run Pipelines** needs to be customized to support **Page View Events**.  The **Pipelines** need to be updated with **Page View Event** specific pipelines.  Duplicate: 

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipelines/xConnect Contacts to Dynamics Sync Pipelines/Process Single Email Sent Event from xConnect Pipeline
```

Name the copy **Process Single Page View Event from xConnect Pipeline**.  Rename the child nodes to refer to **Page View Events**:

* Resolve Dynamics Email by Id > Resolve Dynamics View by Id
* Apply Mapping to set Contact Id to Dynamics Email Id > Apply Mapping to set Contact Id to Dynamics View Id
* Read Message Item from Email Sent Event> Read Page View Item from Page View Event
* Apply Mapping for xConnect Message Item to Dynamics Email > Apply Mapping for xConnect Page View Item to Dynamics View

![Iterate Email Sent Events and Run Pipelines](/images/sitecore-dynamics-custom-activity/iterate-email-sent-events-and-run-pipelines.png "Iterate Email Sent Events and Run Pipelines")

Looking through the child steps, the following steps need to be updated:

* Resolve Dynamics View by ID
* Read Page View Item from Page View Event
* Apply Mapping for xConnect Page View Item to Dynamics View

**Resolve Dynamics View by ID** only needs one change made.  That is updating the **Entity Name** from `email` to `task`.  The entity name is the name of the entity in dynamics that will be populated.  In our case for demonstration purposes, we have chosen the **task** entity.

![Resolve Dynamics View by Id](/images/sitecore-dynamics-custom-activity/resolve-dynamics-view-by-id.png "Resolve Dynamics View by Id")

### Expose PageViewEvent Object for Mapping

Let's address **Read Page View Item from Page View Event** next.  The **Source Object Value Accessor** needs to be updated with a Value Accessor specific to Page View Events.

![Read Message Item from Email Sent Event](/images/sitecore-dynamics-custom-activity/read-message-item-from-email-sent-event.png "Read Message Item from Email Sent Event")

The item referenced in the **Source Object Value Accessor** is:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Accessors/Providers/xConnect/Message Item from Email Sent Event
```

Let's duplicate this item and name it **Page View Item from Page View Event - 1N**.

![Message Item from Email Sent Event](/images/sitecore-dynamics-custom-activity/message-item-from-email-sent-event.png "Message Item from Email Sent Event")

The duplicated item refers the value reader:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Readers/Providers/xConnect/Message Item From Email Sent Event Reader
```

A new value reader should be created to support the Page View Event.  Create a new value reader by duplicating:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Readers/Providers/xConnect/Message Item From Email Sent Event Reader
```

Name the duplicate **Page View Item From Page View Event Reader - 1N**.

![Message Item From Email Sent Event Reader](/images/sitecore-dynamics-custom-activity/message-item-from-email-sent-event-reader.png "Message Item From Email Sent Event Reader")

The value readers refer to a Converter Type of the value:

```
Sitecore.DataExchange.Providers.XConnect.Local.DataAccess.Readers.MessageItemFromEmailSentEventValueReaderConverter, Sitecore.DataExchange.Providers.XConnect.Local
```

We will need to figure out what the existing converter does and create our own for the Page View Event.  After some reflection, I came up with the following to support the Page View Event.

```C#
using Sitecore.DataExchange;
using Sitecore.DataExchange.Attributes;
using Sitecore.DataExchange.Converters;
using Sitecore.DataExchange.DataAccess;
using Sitecore.DataExchange.Repositories;
using Sitecore.Services.Core.Model;

namespace OneNorth.DynamicsPoc.DataExchange.Providers.XConnect.Local.DataAccess.Readers
{
    [SupportedIds(new string[] { "{39B489BD-C876-4FEC-ADD4-85253473891F}" })]
    public class PageViewItemFromPageViewEventValueReaderConverter : BaseItemModelConverter<IValueReader>
    {
        public PageViewItemFromPageViewEventValueReaderConverter(IItemModelRepository repository) : base(repository)
        {
        }

        protected override ConvertResult<IValueReader> ConvertSupportedItem(ItemModel source)
        {
            return ConvertResult<IValueReader>.PositiveResult(new PageViewItemFromPageViewEventValueReader());
        }
    }
}
```

```C#
using Sitecore.DataExchange.DataAccess;
using Sitecore.XConnect.Collection.Model;
using System;

namespace OneNorth.DynamicsPoc.DataExchange.Providers.XConnect.Local.DataAccess.Readers
{
    public class PageViewItemFromPageViewEventValueReader : IValueReader
    {
        public PageViewItemFromPageViewEventValueReader()
        {
        }

        public virtual ReadResult Read(object source, DataAccessContext context)
        {
            PageViewEvent pageViewEvent = source as PageViewEvent;
            if (pageViewEvent == null)
            {
                return ReadResult.NegativeResult(DateTime.Now);
            }
            return ReadResult.PositiveResult(pageViewEvent, DateTime.Now);
        }
    }
}
```

Backtrack and update all references in the duplicated items.

### Setup Mappings

Now we can address the **Apply Mapping for xConnect Page View Item to Dynamics View** pipeline step.  This pipeline step references a Mapping set that needs to be customized for Page View Events.  The mapping set references is:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Value Mapping Sets/xConnect Event to Dynamics Activity Mappings/xConnect EXM Message Item to Dynamics Email Activity
```

![Apply Mapping for xConnect Message Item to Dynamics Email](/images/sitecore-dynamics-custom-activity/apply-mapping-for-xconnect-message-item-to-dynamics-email.png "Apply Mapping for xConnect Message Item to Dynamics Email")

We will need to duplicate:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Value Mapping Sets/xConnect Event to Dynamics Activity Mappings/xConnect EXM Message Item to Dynamics Email Activity
```

Name it **xConnect Page View Item to Dynamics View Activity - 1N**. 

![xConnect EXM Message Item to Dynamics Email Activity](/images/sitecore-dynamics-custom-activity/xconnect-exm-message-item-to-dynamics-email-activity.png "xConnect EXM Message Item to Dynamics Email Activity")

The fields will need to be updated to be specific for *Page View Events*.  For this exercise, the fields we are interested in are:

* Description
* Start Time
* Subject

Each field has a **Source Accessor** and **Target Accessor**.  The targets accessors for Dynamics are already complete.  The correct target accessors can be taken from the **xConnect EXM Message Item to Dynamics Email Activity**.  The following fields can be considered the same from a target accessor perspective.  Copy the Target Accessor from the email to the page view fields.

* Body > Description
* Start Time > Start Time
* Subject > Subject

The source accessors for Page View Events need to be implemented.  Create a new **Value Accessor Set** under:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Accessor Sets/Providers/xConnect
``` 

Name it **Page View Event - 1N**.  Follow the pattern of:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Data Access/Value Accessor Sets/Providers/xConnect/EXM Message Item
```

The only fields we need to create are the ones we want to use.  In our case the following **Property Value Accessors** need to be created:

* Text on Page View Event
* Timestamp on Page View Event
* Url on Page View Event

![Page View Event](/images/sitecore-dynamics-custom-activity/page-view-event-value-accessor-set.png "Page View Event")

The following settings should be used:

* Text on Page View Event
    * Property Name: `Text`
* Timestamp on Page View Event
    * Property Name: `Timestamp`
* Url on Page View Event
    * Property Name: `Url`

The framework pulls the values off the PageViewEvent object using the property names defined above.  These property value accessors can then be referenced within the value mappings under:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Value Mapping Sets/xConnect Event to Dynamics Activity Mappings/xConnect Page View Item to Dynamics View Activity - 1N
```

The following Value mapping settings should be used:

* Description
    * Source Accessor: `Text on Page View Event`
    * Target Accessor: `Description on Dynamics Activity`
* Start Time
    * Source Accessor: `Timestamp on Page View Event`
    * Target Accessor: `Actual Start on Dynamics Activity`
* Subject
    * Source Accessor: `URL on Page View Event`
    * Target Accessor: `Subject on Dynamics Activity`

Go back and update:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Pipelines/xConnect Contacts to Dynamics Sync Pipelines/Process Single Page View Event from xConnect Pipeline - 1N/Apply Mapping for xConnect Page View Item to Dynamics View
``` 

Point mapping set to:

```
/sitecore/system/Data Exchange/Dynamics Tenant/Value Mapping Sets/xConnect Event to Dynamics Activity Mappings/xConnect Page View Item to Dynamics View Activity - 1N
```

![Apply Mapping for xConnect Page View Item to Dynamics View](/images/sitecore-dynamics-custom-activity/apply-mapping-for-xconnect-page-view-item-to-dynamics-view.png "Apply Mapping for xConnect Page View Item to Dynamics View")

## Testing and Troubleshooting

All of the necessary changes have been made to allow Page View Events to migrate into Dynamics Activity Tasks.  To test the integration click **Run Pipeline Batch** on the **xConnect Contacts to Dynamics Sync** pipeline batch.

![xConnect Contacts to Dynamics Sync](/images/sitecore-dynamics-custom-activity/xconnect-contacts-to-dynamics-sync-run-pipeline-batch.png "xConnect Contacts to Dynamics Sync")

You should see Tasks appear under dynamics based on the Page Views that are logged to xConnect.

![Dynamics 365](/images/sitecore-dynamics-custom-activity/dynamics-365.png "Dynamics 365")

For debugging purposes, you can change the log level as seen above.  You can also download the log from the last run.

If you need additional logging, most steps support enabling telemetry.

![Telemetry](/images/sitecore-dynamics-custom-activity/xconnect-contacts-to-dynamics-sync-run-pipeline-batch-2.png "Telemetry")

## Conclusion

As you can see, with a bit of reverse engineering you can customize the **Sitecore Connect for Microsoft Dynamics 365 for Sales** connector and integration.  Hopefully the steps I provided give you an idea of how to do so.  I am new to the Dynamics connector, and this is the way I figured out how to customize it.  I hope you find this article helpful.
