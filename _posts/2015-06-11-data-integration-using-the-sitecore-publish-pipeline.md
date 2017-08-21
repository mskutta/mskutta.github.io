---
layout: post
title: "Data Integration using the Sitecore Publish Pipeline"
date:   2015-06-11 00:00:00 -0500
categories: sitecore
tags: sitecore data integration
author: Mike Skutta
excerpt: I recently used the Sitecore Publish Pipeline to do a data integration with other systems in our network. While going through this process, it occurred to me that a how-to might be helpful to others trying to do the same. We had a requirement where we needed published content to be synced with 3rd party systems in near real-time. The 3rd party system essentially needed the content from the web database. The best way to sync the data in near real-time was to plug into the process that Sitecore used to populate the web database. This would ensure that we would have access to the data as it was being published. Sitecore uses the publish pipeline to migrate published items from the master database to the web database. This would be the perfect spot to plug in our logic.
---

* content
{:toc}

## Overview

I recently used the Sitecore Publish Pipeline to do a data integration with other systems in our network. While going through this process, it occurred to me that a “how-to” might be helpful to others trying to do the same.

We had a requirement where we needed published content to be synced with 3rd party systems in near real-time. The 3rd party system essentially needed the content from the web database. The best way to sync the data in near real-time was to plug into the process that Sitecore used to populate the **web** database. This would ensure that we would have access to the data as it was being published. Sitecore uses the publish pipeline to migrate published items from the **master** database to the **web** database. This would be the perfect spot to plug in our logic.

## Performance and Durability

One thing to watch out for when adding logic to the publish pipeline or other pipelines is the impact to the performance. Each step in a pipeline is synchronous. Adding an additional step or steps that are slow, will slow down the overall process.

Typically when sending data to a 3rd party system, some sort of action needs to be performed to send the data to the system. This might be a call to a web service on the 3rd party system. Adding a call to a web service for each item during the publish process has the potential to really slow down publishing. Not only will there be the latency involved when calling the web service over the network, we may also have to wait for the 3rd party system to process the data before the web service call returns. What happens when the 3rd party system is not available and the web service is not responding? This will not only slow the system down as it waits for the web service calls to time out, but the 3rd party system will miss the data update and will get out of sync.

One way to resolve the issue of performance and durability, is to use a [service bus](http://en.wikipedia.org/wiki/Enterprise_service_bus). Instead of calling the web service directly on the 3rd party system, we would publish a message on the service bus. The 3rd party system would then subscribe to the messages and process them as they became available. The act of publishing messages typically performs very well. Using a service bus, we do not have to wait for the 3rd party system to process the published message. We simply publish the message onto the bus and move on. The 3rd party system will process the message asynchronously when it is ready. Messages on a bus will typically queue up if the subscriber (3rd party system) is not ready to receive them. Once the subscriber is ready, it can process the queued messages. This prevents the subscriber from losing any data.

There are many Service Bus options available. Here are a few:
* [Azure Service Bus](http://azure.microsoft.com/en-us/services/service-bus/)
* [Amazon SQS](http://aws.amazon.com/sqs/)
* [RabbitMQ - .Net](https://www.rabbitmq.com/)

## Implementation

The out of the box configuration for the publish pipeline is as follows:

``` xml
<pipelines>
    <publish help="Processors should derive from Sitecore.Publishing.Pipelines.Publish.PublishProcessor">
        <processor type="Sitecore.Publishing.Pipelines.Publish.OverridePublishContext, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.Publish.AddLanguagesToQueue, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.Publish.AddItemsToQueue, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.Publish.RaiseQueuedEvents, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.Publish.ProcessQueue, Sitecore.Kernel"/>
    </publish>
    <publishItem help="Processors should derive from Sitecore.Publishing.Pipelines.PublishItem.PublishItemProcessor">
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.RaiseProcessingEvent, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.CheckVirtualItem, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.CheckSecurity, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.DetermineAction, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.PerformAction, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.AddItemReferences, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.RemoveUnknownChildren, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.MoveItems, Sitecore.Kernel"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.RaiseProcessedEvent, Sitecore.Kernel" runIfAborted="true"/>
        <processor type="Sitecore.Publishing.Pipelines.PublishItem.UpdateStatistics, Sitecore.Kernel" runIfAborted="true">
          <traceToLog>false</traceToLog>
        </processor>
    </publishItem>
</pipelines>
```

Through discovery, I found the **Sitecore.Publishing.Pipelines.PublishItem.PerformAction** pipeline step did the actual work of updating the **web** database based on the data from the **master** database. For the most part all the information we need to send to the 3rd party system is available to us after the **PerformAction** pipeline step runs. At this point, we know the Operation that was performed on the data. Prior to this step, we do not know what the Operation is. The Operation is used to tell the 3rd party system what to do with the data. The information that is missing at this point is the actual item related to deletions. Deleted items no longer exist after the **PerformAction** pipeline step runs. If we want any information about items that are deleted as part of publishing, we need to tap in before the **PerformAction** pipeline step runs. Because of this, we need to create 2 **PublishItem** pipeline steps to capture all the information we need, one before **PerformAction** and one after.

As I mentioned before, we need a step before the **PerformAction** step to gather the information about items that are about to be deleted. We first need to determine if the **PublishItemContext.Action** is **PublishAction.DeleteTargetItem**. If it is, the item is about to be deleted. We can then attempt to retrieve the item that is about to be deleted from the publishing target. If it is not there, we can try the publishing source.

> Note: When attempting to retrieve the item, if the item is not found, the item has already been deleted. This can occur when more than one language is published. The first language published will delete the item.

Once we have the item we can add it to the **PublishItemContext.CustomData** for use further down the pipeline. The PublishItemContext.CustomData is shared between all the pipeline steps. Here is the complete code:

``` csharp
public class EvaluateAction : PublishItemProcessor
{
    public override void Process(PublishItemContext context)
    {
        Assert.ArgumentNotNull(context, "context");

        // We just want to process deletes because this is the only time the item being deleted may exist.
        if (context.Action != PublishAction.DeleteTargetItem && context.Action != PublishAction.PublishSharedFields)
            return;

        // Attempt to find the item.  If not found, item has already been deleted. This can occur when more than one langauge is published. The first language will delete the item.
        var item = context.PublishHelper.GetTargetItem(context.ItemId) ??
                   context.PublishHelper.GetSourceItem(context.ItemId);

        if (item == null) 
            return;

        // Hold onto the item for the EvaluateResult PublishItemProcessor.
        context.CustomData.Add("Item", item);
    }
}
```

Now that we have a way to get deleted items, we can move on to the pipeline step that occurs after the **PerformAction** step. We don't want to send unnecessary data to the 3rd party system. If the **PublishOperation** is **Skipped** or **None**, that means Target (typically the Web database) and the 3rd party system has already been updated. We don't want to send this data again. If a republish is performed we always want to process the delete actions and we don't want to rely on the **PublishItemContext.Result.Operation** for this. Republish can be used as a way to true-up the data in the target database and the 3rd party system. This may be necessary if the data becomes out of sync.

``` csharp
if ((context.Action != PublishAction.DeleteTargetItem || context.PublishOptions.CompareRevisions) &&
    (context.Result.Operation == PublishOperation.Skipped ||
     context.Result.Operation == PublishOperation.None)) 
    return;
```

> Note on Deletes... The PublishOperation of Deleted will only come though once for one of the published languages. The other languages will have a result of none. As a result, we only publish one Delete message regardless of the number of published languages

Next, we want to get the item related to the Action and Operation. If the item is due to a delete, we need to pull it out of the **PublishItemContext.CustomData** from the previous step. Otherwise we get the current item from **PublishItemContext.VersionToPublish**. To prevent extra noise, you may want to only process items that are of a certain template type. We really only want to process items that the 3rd party system cares about. We also need to ignore the __Standard Values items. We can detect these if the TemplateID is the same as the ParentID.

``` csharp
var item = (Item)context.CustomData["Item"] ?? context.VersionToPublish;
if (item == null)
    return;
    
var template = TemplateManager.GetTemplate(item);
if (!_templateIdsToPublish.Any(x => template.ID == new ID(x)))
    return;

// Don't publish messages for standard values
if (item.ParentID == item.TemplateID)
    return;
```

We need to take the information from the Item and operation and publish it onto the Service Bus for consumption by the 3rd party system. First we need to define the messages that we will be publishing on the bus. There are three primary messages we will be publishing; Created, Updated and Deleted. These map to the Publish Operations that were performed. We want to include the data from the item that was impacted as part of the messages. All of this must be serializable for sending over the message bus. Because of this we will be using Plain Old CLR Objects or POCOs to send the data. We can use a tool such as Glass Mapper to map the data from a Sitecore item to the model objects or entities. First, we need to create the messages that will contain the model objects / entities.

``` csharp
public interface IEvent
{
    Guid Id { get; set; }
}

[Serializable]
public class CreatedEvent<TEntity> : IEvent
    where TEntity : class
{
    public Guid Id { get; set; }
    public TEntity Entity { get; set; }
}

[Serializable]
public class UpdatedEvent<TEntity> : IEvent
    where TEntity : class
{
    public Guid Id { get; set; }
    public TEntity Entity { get; set; }
}

[Serializable]
public class DeletedEvent<TEntity> : IEvent
    where TEntity : class
{
    public Guid Id { get; set; }
    public TEntity Entity { get; set; }
}
```

Now we need to use the messages and mappings to send the data over the service bus. The subscribers/consumers of the messages can take action based on the message type and the Entity stored in the message.

``` csharp
// Convert item to model object / entity / dto
var entity = _mapper.Map(item); // This is dependent on the mapper you decide to use.
if (entity == null)
    return;
var entityType = entity.GetType();

// Create the message we want to send on the service bus
var operation = (context.Action == PublishAction.DeleteTargetItem && !context.PublishOptions.CompareRevisions) ? PublishOperation.Deleted : context.Result.Operation;
if (operation == PublishOperation.Created)
    messageType = typeof(CreatedEvent<>).MakeGenericType(entityType);
if (operation == PublishOperation.Updated)
    messageType = typeof(UpdatedEvent<>).MakeGenericType(entityType);
if (operation == PublishOperation.Deleted)
    messageType = typeof(DeletedEvent<>).MakeGenericType(entityType);
if (messageType == null)
    return;
    
// Add the entity to the message
var publishEvent = Activator.CreateInstance(messageType);
((IEvent)publishEvent).Id = context.ItemId.Guid;
var property = messageType.GetProperty("Entity");
if (property != null)
    property.SetValue(publishEvent, entity, null);

// Publish the event on the service bus
_bus.Publish(publishEvent); // This is dependent on the service bus you decide to use.
```

For completeness sake, I have included the full source for the EvaluateResult pipeline step below. The _mapper and _bus dependencies are dependent on the implementation. These would typically be injected as dependencies. The code is included as a single method for readability. It is best to break the code up into smaller methods.

``` csharp
public class EvaluateResult : PublishItemProcessor
{
    public override void Process(PublishItemContext context)
    {
        Assert.ArgumentNotNull(context, "context");

        // We want to rely on smart publishing so we don't send unnecessary data.  Deletes are a weird sceneiro. 
        // If a Republish is performed, we need the delete actions to come through.  We cant rely on the Result.Operation for this.
        if ((context.Action != PublishAction.DeleteTargetItem || context.PublishOptions.CompareRevisions) &&
            (context.Result.Operation == PublishOperation.Skipped ||
             context.Result.Operation == PublishOperation.None)) 
            return;

        // Note on Deletes...  The Operation of delete will only come though once for one of the published languages.  
        // The other languages will have a result of none.  As a result, we only publish one Delete message regardless of the number of published languages 

        // Note on HideVersion = true...  The operation of a smart publish will always be PublishOperation.Updated.  The causes unnecessary processing.

        // Dont publish a message if we are not interested in this item based on the base template.  This check is faster than performing a service bus publish and checking later.
        // For deletes the VersionToPublish is the parent, we need to get the item from the EvaluateAction step
        var item = (Item)context.CustomData["Item"] ?? context.VersionToPublish;
        if (item == null)
            return;
            
        var template = TemplateManager.GetTemplate(item);
        if (!_templateIdsToPublish.Any(x => template.ID == new ID(x)))
            return;

        // Don't publish messages for standard values
        if (item.ParentID == item.TemplateID)
            return;

        // Convert item to model object / entity / dto
        var entity = _mapper.Map(item); // This is dependent on the mapper you decide to use.
        if (entity == null)
            return;
        var entityType = entity.GetType();

        // Create the message we want to send on the service bus
        var operation = (context.Action == PublishAction.DeleteTargetItem && !context.PublishOptions.CompareRevisions) ? PublishOperation.Deleted : context.Result.Operation;
        if (operation == PublishOperation.Created)
            messageType = typeof(CreatedEvent<>).MakeGenericType(entityType);
        if (operation == PublishOperation.Updated)
            messageType = typeof(UpdatedEvent<>).MakeGenericType(entityType);
        if (operation == PublishOperation.Deleted)
            messageType = typeof(DeletedEvent<>).MakeGenericType(entityType);
        if (messageType == null)
            return;
            
        // Add the entity to the message
        var publishEvent = Activator.CreateInstance(messageType);
        ((IEvent)publishEvent).Id = context.ItemId.Guid;
        var property = messageType.GetProperty("Entity");
        if (property != null)
            property.SetValue(publishEvent, entity, null);

        // Publish the event on the service bus
        _bus.Publish(publishEvent); // This is dependent on the service bus you decide to use.
    }
}
```

Lastly, we need to register the pipeline steps via configuration.

``` xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <pipelines>
      <publishItem>
        <processor patch:before="*[@type='Sitecore.Publishing.Pipelines.PublishItem.PerformAction, Sitecore.Kernel']" type="OneNorth.Integration.ServiceBus.Pipelines.PublishItem.EvaluateAction, OneNorth.Integration.ServiceBus"/>
        <processor patch:after="*[@type='Sitecore.Publishing.Pipelines.PublishItem.PerformAction, Sitecore.Kernel']" type="OneNorth.Integration.ServiceBus.Pipelines.PublishItem.EvaluateResult, OneNorth.Integration.ServiceBus"/>
      </publishItem>
    </pipelines>
  </sitecore>
</configuration>
```

## Conclusion

I hope someone finds this to be helpful when designing a near real-time integration.