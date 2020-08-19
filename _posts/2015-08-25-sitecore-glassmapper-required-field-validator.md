---
layout: post
title: "Sitecore: Glass.Mapper Required Field Validator"
date:   2015-08-25 00:00:00 -0500
categories: sitecore
tags: sitecore glass.mapper
author: Mike Skutta
excerpt: Not too long ago I released the Express Subitem Sitecore Marketplace module. This module allows content editors to edit multiple child Sitecore items at the same time within the context of the parent item. This field is commonly used to manage lists of related items that exist only within the context of the parent.
---

* content
{:toc}

## Overview

Not too long ago I released the [Express Subitem](https://marketplace.sitecore.net/Modules/E/Express_Subitem.aspx?sc_lang=en) Sitecore Marketplace module. This module allows content editors to edit multiple child Sitecore items at the same time within the context of the parent item. This field is commonly used to manage lists of related items that exist only within the context of the parent.

The example I gave was one where a Person record contained education history. For this article, let's assume the education sub-item has a Drop Link to choose a school from a defined list.

![Express Subitem](/images/sitecore-glassmapper-required-field-validator/sitecore_glass_mapper_1.jpg)

In this particular example, we want the School field to be required seeing as it does not make sense to have an education record without a school. We can easily set the School to be a required field through validation rules, meaning a content administrator must pick a school or they will not be able to save the record.

It's important to note that there is a problem when making relationship fields required. Making a relationship field required does not guarantee the related item exists on the live site. A content administrator can chose a School that is not publishable or published. If the person with the education record is published (publish item and subitems), the school may be missing in the web database. This has the effect of a required field missing a value.

In the case where the school is missing, we don&rsquo;t want to display the education record on the live site. This would typically be accomplished by adding custom logic to hide the education record if the school was missing. This is error prone as this can be missed and not discovered until the situation arises on the live site.

It would be beneficial to handle this situation without the developer having to think about each situation when this could occur.

> Note: This situation can apply to all relationships that are setup as required fields. This does not just apply to the Express Subitem module. The Express Subitem module was used as an example.

## Solution

Many Sitecore implementations use the popular [Glass.Mapper ORM](http://www.glass.lu/) to render content. Continuing the example, we can use Glass.Mapper to render the Person, including Education records. These examples use Glass.Mapper 4.0.

*Person.cshtml*
``` cshtml
@model IPerson

<p>
    <label>First Name:</label> @Model.FirstName<br/>
    <label>Last Name:</label> @Model.LastName<br/>
</p>

<h1>Education</h1>
@foreach (var item in Model.Education)
{
    <p>
        <label>School:</label> @item.School.Name<br/>
        <label>Year:</label> @item.YearOfDegree<br/>
        <label>Degree:</label> @item.Degree<br/>
    </p>
}
```

Logic would typically be added around the iteration of the Model.Education to ignore records that do not have a School. Instead of adding custom logic on render, we can tap into the flexibility of Glass.Mapper to return only valid records. Glass.Mapper provides a pipeline framework where the out of the box behavior can be modified by plugging in custom pipeline steps.

First, we need to tell Glass.Mapper what the required fields are. We can do this through a custom property attribute. This attribute can be applied to all required properties on the Glass objects.

*SitecoreRequiredAttribute.cs*
``` csharp
[AttributeUsage(AttributeTargets.Property)]
public class SitecoreRequiredAttribute : Attribute
{

}
```

Next, we need to figure out where to put our custom pipeline logic. The ObjectConstructionFactory handles building the objects returned in the ORM. We can have the ObjectConstructionFactory return null when a required field is missing. To plug into the ObjectConstructionFactory pipeline, we modify the CreateResolver method in the GlassMapperScCustom.cs class. We want our pipeline step to run before the resulting object is constructed. We also want it to happen after the cache is checked so it only runs when necessary. The task will be inserted just after the CacheCheckTask. The implementation of our task will be done in the ValidateItemTask class.

*GlassMapperScCustom.cs*
``` csharp
public static IDependencyResolver CreateResolver() 
{
    var config = new Glass.Mapper.Sc.Config();

    var resolver = new DependencyResolver(config);

    // Add ValidateItemTask just after the CacheCheckTask
    var cacheCheckTaskIndex = resolver.ObjectConstructionFactory.GetItems().ToList().FindIndex(x => x is CacheCheckTask);
    resolver.ObjectConstructionFactory.Insert(cacheCheckTaskIndex + 1, () => new ValidateItemTask());

    return resolver;
}
```

Now, we will create the ValidateItemTask. All tasks that participate in the ObjectConstructionFactory implement the IObjectConstructionTask interface. The Execute method must be implemented. Within this method, the args.Result can be set to null if the object is not valid. Args.AbortPipeline should also be called in this case so further tasks are not called to resolve the object. To prevent issues with data loaders and previewing, the validation logic will only run when outside the context of the master database, i.e. the web database.

*ValidateItemTask.cs*
``` csharp
 public class ValidateItemTask : IObjectConstructionTask
  {
      public void Execute(ObjectConstructionArgs args)
      {
          // Do not validate in the context of the master database
          var service = (SitecoreService)args.Service;
          if (service.Database.Name == "master")
              return;

          // Validate fields.
          if (!IsValid(args))
          {
              // Item is not valid, set the result to empty.
              args.Result = null;
              args.AbortPipeline();
          }
      }
  }
```

This IsValid method needs to be implemented, validating the context item based on the SitecoreRequiredAttribute. It will check the values of all of the fields marked with this attribute. If the field is empty, false will be returned. We will first create a helper methods to return the list of required fields for a given context. The GetRequiredFields helper method uses reflection to obtain the attributes on the configured properties. The results are cached in a static variable.

ValidateItemTask.cs
``` csharp
public class ValidateItemTask : IObjectConstructionTask
{
    private static readonly ConcurrentDictionary<Type, SitecoreFieldConfiguration[]> _requiredFields = new ConcurrentDictionary<Type, SitecoreFieldConfiguration[]>();

    private SitecoreFieldConfiguration[] GetRequiredFields(ObjectConstructionArgs args)
    {
        var type = args.Configuration.Type;
        return _requiredFields.GetOrAdd(type, x => RequiredFieldsValueFactory(args));
    }

    private SitecoreFieldConfiguration[] RequiredFieldsValueFactory(ObjectConstructionArgs args)
    {
        return args.Configuration.Properties
            .Where(property => Attribute.IsDefined(property.PropertyInfo, typeof(SitecoreRequiredAttribute), true))
            .OfType<SitecoreFieldConfiguration>()
            .ToArray();
    }
}
```

Next, the IsValid method can be implemented. It will first get the required fields using the helper methods from above. Then, for each field, it will check the validity. If a field is not valid, it returns false. In most cases, if the value of the field is null or empty, we can return false. There are a few cases where the field may be populated, but it is still empty. Relationship fields may have an ID present, but the item associated with the ID has not been published, and therefore is not there. For these fields we need to perform some extra checks. In this particular case, we just need to check for the presence of the item in the database. I did not fill in all of the logic for the different field types. This can be an exercise for another time.

*ValidateItemTask.cs*
``` csharp
public class ValidateItemTask : IObjectConstructionTask
{
    private bool IsValid(ObjectConstructionArgs args)
    {
        var context = (SitecoreTypeCreationContext) args.AbstractTypeCreationContext;
        var service = (SitecoreService)args.Service;

        var requiredFields = GetRequiredFields(args);
        foreach (var requiredField in requiredFields)
        {
            var field = Utilities.GetField(context.Item, requiredField.FieldId, requiredField.FieldName);

            // If the field does not exist it is not valid.
            if (field == null)
                return false;

            // If the field value is empty it is not valid.
            var value = field.GetValue(true, true);
            if (string.IsNullOrEmpty(value))
                return false;

            // Perform type specific checks.
            switch (field.Type)
            {
                case "Checkbox":
                case "Date":
                case "Datetime":
                case "Integer":
                case "Number":
                case "Multi-Line Text":
                case "Password":
                case "Rich Text":
                case "Single-Line Text":
                    break;
                case "General Link":
                case "General Link with Search":
                    {
                        var linkField = ((LinkField) field);
                        if (linkField.IsInternal)
                        {
                            var item = linkField.TargetItem;
                            if (item == null || item.Versions.Count == 0)
                                return false;
                        }
                        else if (linkField.IsMediaLink)
                        {
                            var item = linkField.TargetItem;
                            if (item == null)
                                return false;
                        }
                        break;
                    }
                case "Grouped Droplink":
                case "Droplink":
                case "Droptree":
                    {
                        ID itemId;
                        if (!ID.TryParse(value, out itemId))
                            return false;
                        var item = service.Database.GetItem(itemId);
                        if (item == null || item.Versions.Count == 0)
                            return false;
                        break;
                    }
                case "Multilist":
                case "Multilist with Search":
                case "Checklist":
                case "Searchable Tree List":
                case "Treelist":
                case "TreelistEx":
                case "ItemAggregate":
                    {
                        var itemIds = ID.ParseArray(value);
                        var exists = itemIds.Select(itemId => service.Database.GetItem(itemId))
                               .Any(item => item != null && item.Versions.Count != 0);
                        if (!exists)
                            return false;
                        break;
                    }
                case "File":
                    {
                        FileField fileField = field;
                        if (fileField.MediaItem == null)
                            return false;
                        break;
                    }
                case "Image":
                    {
                        ImageField imageField = field;
                        if (imageField.MediaItem == null)
                            return false;
                        break;
                    }
                default:
                    throw new NotImplementedException(string.Format("Support for the field type '{0}' has not been implemented.", field.Type));
            }
        }
        return true;
    }
}
```

Give it a try. All required relationship fields that are missing the related item in the web database will cause the request for the item/object to return null.
 
## Conclusion

Glass.Mapper is very flexible. With this flexibility, we were able to add business logic to the pipeline. This business logic eliminates the need of adding additional checks when required fields are missing.
