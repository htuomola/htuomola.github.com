---
layout: post
title: "Allowing null foreign key values in Linq-to-SQL"
date: 2011-09-28 17:50
comments: true
categories: 
 - windows phone
 - programming
 - .net
---

This is quite simple finally but it took me a while figure it out. So while building the Metropas application I got the following exception when trying to save a new Favorite object to the database.

![Linq to SQL null foreign key exception](/blog/images/allowing-null-foreign-key-values-in-linq-to-sql_1.png)

Exception was: 

    SQLCeException: A foreign key value cannot be inserted     
    because a corresponding primary key value does not exist. 
    [Foreign key constraint name = FK_Place_Location ].

Before proceeding any further, **read the error message carefully**. Then read it again. The last word denotes the table and column that this relates to, in this case column Location in table Place. It took me a while as I haphazardly misread the exception and was looking for the problem in the from table/object :-).

<!--more-->

So the first point here is to check if Place.Location really is null. In my case it was and that was perfectly fine (business logic –wise) but apparently SQL Compact thought otherwise. Here’s the relevant declarations from code:

{% codeblock lang:csharp %}
[Column]
internal int LocationId;

private EntityRef<GeoLocation> _location;
 
[Association(Storage = "_location", ThisKey = "LocationId", 
	OtherKey = "Id", IsForeignKey = true)]
public GeoLocation Location
{% endcodeblock %}
	
Next, I searched for the MSDN documentation, Google and Stack Overflow on Linq2Sql and nullable foreign keys without finding anything relevant. I expected to find something like a property “Nullable” from [AssociationAttribute][assocAttr]. Nope. All I could find was [ColumnAttribute.CanBeNull][colAttrCanBeNull] property but my problem was that the Location field has an Association attribute, not column. Then I realized that there’s the LocationId field that is used to store the Id of the referenced table. I had completely forgotten it as it’s more like “necessary boilerplate code” than often used data.

Obviously the id field should be able to be null so I applied the `“CanBeNull=true”` attribute to my LocationId attribute and redeployed the app. Still, I got an SQLException of a different kind (which I wasn’t able to reproduce for this blog post). Basically it said that LocationId could not contain a null value. Thinking again (yeah, this wasn’t my brightest moment) I realized that as the field is of type int, it doesn’t allow nulls. So I defined it as nullable integer and redeployed the app again. This time it worked! While testing it some more I noticed that the CanBeNull=true declaration seems to be unnecessary once the field is defined as nullable. So the complete code for a nullable foreign key property is:
{% codeblock lang:csharp %}
[Column]
internal int? LocationId;
 
private EntityRef<GeoLocation> _location;
 
[Association(Storage = "_location", ThisKey = "LocationId", 
	OtherKey = "Id", IsForeignKey = true)]
public GeoLocation Location
{
	get { return _location.Entity; }
	set
	{
		RaisePropertyChanging("Location");
		_location.Entity = value;
 
		if (value != null)
			LocationId = value.Id;
 
		RaisePropertyChanged("Location");
	}
}
{% endcodeblock %}

Hope this helps someone who’s wondering the same thing.

P.S. Keep in mind that the Local database on Windows Phone 7 is stored in isolated storage, which is emptied always when the emulator is restarted. So if you are doing modifications to the database schema and your application doesn’t handle those (see [MSDN: Local Database Migration Overview for Windows Phone][msdnLocalDbMigration]), it might be best to just restart the emulator in case of problems to force recreation of the database.

[msdnLocalDbMigration]: http://msdn.microsoft.com/en-us/library/hh394018(v=VS.92).aspx
[colAttrCanBeNull]: http://msdn.microsoft.com/en-us/library/system.data.linq.mapping.columnattribute.canbenull(v=VS.95).aspx
[assocAttr]: http://msdn.microsoft.com/en-us/library/system.data.linq.mapping.associationattribute(v=VS.95).aspx