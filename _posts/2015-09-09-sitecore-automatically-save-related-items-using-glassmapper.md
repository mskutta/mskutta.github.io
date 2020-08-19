---
layout: post
title: "Sitecore: Automatically Save Related Items Using Glass.Mapper"
date:   2015-09-09 00:00:00 -0500
categories: sitecore
tags: sitecore glass.mapper
author: Mike Skutta
excerpt: Sitecore items support relationships, be that parent, child or linked, to other items. Glass.Mapper easily works with these types of relationships. It not only supports them when retrieving items, but also while saving. Glass.Mapper even checks to make sure all related items exist before saving the item, essentially enforcing referential integrity.
---

* content
{:toc}

## Overview

Sitecore items support relationships, be that parent, child or linked, to other items. Glass.Mapper easily works with these types of relationships. It not only supports them when retrieving items, but also while saving. Glass.Mapper even checks to make sure all related items exist before saving the item, essentially enforcing referential integrity.

Let's say you have a person's data template that has a related education field. The related education field allows choosing one or more education records. The education data template has the school, year of graduation and degree. On a clean/empty content tree, you could use Glass.Mapper to first create and save the education records. Then, you can create and save the person related to the newly created education records. An example of this may be as follows:

*PersonEducationSaveIndividual.cs*
``` csharp
[SitecoreType(TemplateId = "004c9474-1df4-47ab-9414-c372b25cdde6", AutoMap = true)]
public partial class Education
{
	public virtual Guid Id { get; set; }
	public virtual System.String Name { get; set; }
	
	public virtual string Degree { get; set; }
	public virtual string School { get; set; }
	public virtual System.Int32 YearOfDegree { get; set; }
}

[SitecoreType(TemplateId = "dbb7580b-4702-4f83-911a-b15cad166d5b", AutoMap = true)]
public partial class Person
{
	public virtual Guid Id { get; set; }
	public virtual System.String Name { get; set; }
	
	public virtual System.String FirstName { get; set; }
	public virtual System.String LastName { get; set; }
	public virtual IEnumerable<Education> Education { get; set; }
}

var service = new SitecoreService("master");

var education = new Education();
education.Name = "University of Illinois";
education.School = "University of Illinois";
education.YearOfDegree = 1999;
education.Degree = "BS, Mechanical Engineering";
using (new SecurityDisabler())
{
    service.Create(<parent>, education);
}

var person = new Person();
person.Name = "Mike Skutta";
person.FirstName = "Mike";
person.LastName = "Skutta";
person.Education = new List<Education>{education};
using (new SecurityDisabler())
{
    service.Create(<parent>, person);
}
```

One thing you may have noticed in this example is that the education record cannot really be reused between other people. The data is specific to the person. You could also say the education data is owned by the person. The education record really does not exist outside of the context of the person.

Another thing you may have noticed is that I didn&rsquo;t specify the &lt;parent&gt; when saving the items. If an education record exists only within the context of the person, we could save the education record as a child item of the person. If the person node in the Sitecore content tree is deleted, the child education nodes will also be deleted.

If the education records are owned by the person (parent) and are considered part of the parent, could we treat them as one unit? Could Glass.Mapper handle saving everything in one call? Here is an example of how we could perform a save using Glass.Mapper:

*PersonEducationSaveWhole.cs*
``` csharp
var service = new SitecoreService("master");

var person = new Person();
person.Name = "Mike Skutta";
person.FirstName = "Mike";
person.LastName = "Skutta";
person.Education = new List<Education> { new Education
  {
    Name = "University of Illinois",
    School = "University of Illinois",
    YearOfDegree = 1999,
    Degree = "BS, Mechanical Engineering"
  }
};

using (new SecurityDisabler())
{
    service.Create(<parent>, person);
}
```

Out of the box, Glass.Mapper does not support this approach. Let&rsquo;s see how we can achieve this.

## Solution

Glass.Mapper has flexible pipelines that allow you to inject and replace logic. We can use a custom pipeline task to achieve what we need.

First, we need to tell Glass.Mapper what relationship fields we should be treating as part of the parent. We can do this through a custom property attribute. This attribute can be applied to all required properties on the Glass objects. When a child item is considered part of the parent, it is sometimes referred to as a [Composite Relationship](https://en.wikipedia.org/wiki/Class_diagram#Composition). This attribute can be named SitecoreCompositeRelationshipAttribute.

*SitecoreCompositeRelationshipAttribute.cs*
``` csharp
[AttributeUsage(AttributeTargets.Property)]
public class SitecoreCompositeRelationshipAttribute : Attribute
{
}
```

Next, we need to figure out where to put our custom pipeline logic. The ObjectSavingFactory handles saving the objects in Glass.Mapper. Through trial and error, I discovered that I needed to save the child items before the parent is saved. Glass.Mapper ensures that all related items exist in the database before saving the parent item. Our custom task needs to run before the other ObjectSavingFactory tasks. To plug into the ObjectSavingFactory pipeline, we modify the CreateResolver method in the GlassMapperScCustom.cs class. This task will be inserted before the other tasks. We will name our task the CompositeRelationshipSavingTask.

*GlassMapperScCustom.cs*
``` csharp
public static IDependencyResolver CreateResolver(){
  var config = new Glass.Mapper.Sc.Config();

  var resolver = new DependencyResolver(config);

  // Add the ability to save composite relationships.  This needs to occur before the parent is saved so all IDs are known.
  resolver.ObjectSavingFactory.Insert(0, () => new CompositeRelationshipSavingTask());

  return resolver;
}
```

Now, we will create the CompositeRelationshipSavingTask. All tasks that participate in the ObjectSavingFactory implement the IObjectSavingTask interface. The Execute method must be implemented. Within this method, we will find all properties that have the SitecoreCompositeRelationshipAttribute set. We will then retrieve the value from the attributed properties. The value should be an IEnumerable. We will enumerate each item, creating or saving each child item using Glass.Mapper. To promote consistency, all child items will be placed under a subfolder below the parent item. The subfolder will be named the same name as the Field name with which the property is associated. We will essentially have a subfolder for each Composite Relationship, where the subfolder matches the field name. Now that all items of a Composite Relationship field are in the same sub-folder, we can perform extra operations on the items in the subfolder, such as deleting the children that are no longer being referenced by the parent.

*CompositeRelationshipSavingTask.cs*
``` csharp
public class CompositeRelationshipSavingTask : IObjectSavingTask
{
    public void Execute(ObjectSavingArgs args)
    {
        // Only perform processing if composite relationship fields exist
        var compositeRelationshipFields = GetCompositeRelationshipFields(args);
        if (!compositeRelationshipFields.Any())
            return;

        // Get access to the SitecoreService
        var service = (SitecoreService) args.Service;

        // Get reference to the item being saved
        var savingContext = (SitecoreTypeSavingContext)args.SavingContext;
        var item = savingContext.Item;

        foreach (var compositeRelationshipField in compositeRelationshipFields)
        {
            // Make sure the field is IEnumerable<>
            var propertyType = compositeRelationshipField.PropertyInfo.PropertyType;
            if (!propertyType.GetInterfaces().Contains(typeof(IEnumerable)))
                continue;

            // Make sure the parent folder exists.  The folder is named based on the Field Name.
            var parent = item.Children.FirstOrDefault(x => x.Name == compositeRelationshipField.FieldName);
            if (parent == null)
            {
                parent = item.Add(compositeRelationshipField.FieldName, new TemplateID(TemplateIDs.Folder));
                var displayName = item.Fields[compositeRelationshipField.FieldName].DisplayName;
                using (new EditContext(parent))
                {
                    parent.Appearance.DisplayName = displayName;
                    parent.Appearance.Hidden = true;
                }
            }

            // Get the enumerable field value;
            var enumerable = (IEnumerable)compositeRelationshipField.PropertyInfo.GetValue(args.Target, null);
            if (enumerable == null)
                continue;

            // Enumerate the values, saving them.  Store the IDs created/updated for future use when cleaning up
            var ids = new List<ID>();
            foreach (var child in enumerable)
            {
                var typeConfiguration = args.Context.GetTypeConfiguration<SitecoreTypeConfiguration>(child, false, true);
                var id = typeConfiguration.GetId(child);
                if (ID.IsNullOrEmpty(id) || typeConfiguration.ResolveItem(child, service.Database) == null)
                    service.Create(parent, child);
                else
                    service.Save(child);
                ids.Add(typeConfiguration.GetId(child));
            }

            // Find the differences between the new ids and the old.  Delete the differences
            // Note: the "item" has not been saved yet, so it till has the old values.
            var field = item.Fields[compositeRelationshipField.FieldName];
            var fieldValue = field.Value;
            var oldIds = (!string.IsNullOrEmpty(fieldValue))
                             ? ID.ParseArray(fieldValue)
                             : new ID[0];
            var idsToDelete = oldIds.Except(ids);

            var itemsToDelete = idsToDelete.Select(idToDelete => service.Database.GetItem(idToDelete))
                                           .Where(itemToDelete => itemToDelete != null);
            foreach (var itemToDelete in itemsToDelete)
                itemToDelete.Delete();
        }
    }
}
```

The GetCompositeRelationshipFields method needs to be implemented. It uses reflection to obtain the attributes on the configured properties. The results are cached in a static variable.

*CompositeRelationshipSavingTask.cs*
``` csharp
public class CompositeRelationshipSavingTask : IObjectSavingTask
{
    private static readonly ConcurrentDictionary<Type, SitecoreFieldConfiguration[]> _compositeRelationships = new ConcurrentDictionary<Type, SitecoreFieldConfiguration[]>();

    private SitecoreFieldConfiguration[] GetCompositeRelationshipFields(ObjectSavingArgs args)
    {
        var type = args.SavingContext.Config.Type;
        return _compositeRelationships.GetOrAdd(type, x => CompositeRelationshipsValueFactory(args));
    }

    private SitecoreFieldConfiguration[] CompositeRelationshipsValueFactory(ObjectSavingArgs args)
    {
        return args.SavingContext.Config.Properties
            .Where(property => Attribute.IsDefined(property.PropertyInfo, typeof(SitecoreCompositeRelationshipAttribute), true))
            .OfType<SitecoreFieldConfiguration>()
            .ToArray();
    }
}
```

Give it a try! All attributed relationship fields will be saved along with the parent.

## Conclusion

Glass.Mapper is very flexible. With this flexibility, we were able to add custom logic to the saved pipeline. This logic automatically saves any related item where the relationship field is marked with the SitecoreCompositeRelationshipAttribute.