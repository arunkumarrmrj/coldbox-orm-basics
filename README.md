# Introduction

Object-Relational Mapping (ORM) allows you to work with objects and have them saved to the database automatically. It can greatly simplify create-read-update-delete (CRUD) operations and make your code more object-oriented. Under the hood ColdFusion uses the industry leading ORM framework called Hibernate.

# Configuration

## Application.cfc

Two settings are needed in the Application.cfc file to make an application use ORM: a datasource and ormEnabled set to true. 

The cfartgallery datasource, which uses Derby DB, is installed with the developer version of ColdFusion. ORM will work with any modern database.

Other settings for ORM are set in a structure called ormSettings. In the example below we are setting logsql to true. When set to true the SQL created by Hibernate will display in the ColdFusion debugging information. There are settings for cfclocation to specify where the applications persistent cfc's are placed, and dbcreate which will automatically keep changes made to your model and the database.

The final setting, "invokeImplicitAccessor", we will get to later.

<pre>
component {

this.datasource = "cfartgallery";
this.ormEnabled = true;
this.ormSettings = { logsql : true };
this.invokeImplicitAccessor = true;

}
</pre>

## Entities

An entity is a class and (in basic operations) maps to a single database table. In this first example we create an entity that will be referenced as art in our code and will map to the art table in the artgallery database. The properties map to the columns in the table and can be a subset of all columns as well as having other properties that are not persistent and marked with the attribute persistent=false.

Each persistent cfc must have a primary key and this is denoted with fieldtype="id" (for composite primary keys use multiple properties with fieldtype="id"). Most of the time the primary key will have a database auto-generated primary key in this case we are using the Hibernate powered generator.

Each property will have getters and setters automatically. Both getters and setters can be overwritten. Our client has asked that the art name always be in uppercase, rather than adjust everywhere this is displayed, the example below overwrites the getter to do so. Other functions can be added to the entity as well. The example below has a getProfit function that when a piece is sold calculates the amount of profit. 

The property name maps to a database column via the column attribute. When the column attribute is not provided ColdFusion uses the value in the name as the column.

### art.cfc
<pre>
component persistent="true" {

property name="id" column="artid" fieldtype="id" generator="increment";
property name="name" column="artName" ormtype="string";     
property name="description" ormtype="text";
property name="price" ormtype="double";
property name="isSold" ormtype="boolean";  

property name="artist" fieldtype="many-to-one" cfc="artist";    

function getName() {
  return uCase( variables.name );
}

function getProfit() {
  if ( getIsSold() ) {
    return getPrice() * 0.2;
  } else {
    return 0;
  }
}

}
</pre>

#### Relationships
The final property establishes a relationship between art and artist in this case a many-to-one. It is similar to a foreign-key relationship in a database between two tables except now its between two objects. The artist entity also has a one-to-many relationship to art. Other relationship types include one-to-one and many-to-many although its generally better to use two one-to-many relationships than many-to-many.

ColdFusion automatically adds functions to objects for controlling relationships. For the art many-to-one these are; getArtist(), hasArtist(), setArtist(). For other relationships replace "artist" with the relationship name. getArtist will return an array of objects of the artist, hasArtist is Boolean determining whether there is a artist associated with the art record and setArtist takes an object and sets it to the piece of art.

The artist one-to-many will create; addArt(), getArt(), hasArt(), removeArt(), setArt(). The two new functions are due to a one-to-many, having, well the options for many objects. addArt take a art object and adds it to the relationship, and likewise removeArt will remove an art object from the relationship.

The art relationship below also has a cascade property with the value of "delete". When an artist is deleted this will find all art entities related to it and delete them. While useful this can cause unintended consequences so use carefully, look at the other options for cascade and don't forget you can leave it off all together.


### artist.cfc
For artist we specify the table with the table attribute. One advantage of ORM is the ability to change database table and column names to more programmer friendly terms!

<pre>
component persistent="true" table="ARTISTS" {

property name="id" column="ARTISTID" fieldtype="id" generator="increment";
property name="firstname" ormtype="string";    
property name="lastname" ormtype="string";    

property name="art" fieldtype="one-to-many" cfc="art" fkcolumn="ARTISTID" cascade="delete";      

}
</pre>

## Retrieving Data
To get data there are two functions we can use entityLoad and ormExecuteQuery. First lets use entityLoad() which has many signatures. In the example here, the first argument is the entity, the second a structure of values to match exactly (to get all pass an empty structure), the third what to order data by.
<pre>
&lt;cfset artists = entityLoad( "artist", { firstname: "Jeff" }, "lastname" )&gt;
</pre>

This will return an array of objects where the artists first name is Jeff.

## Outputting

<pre>

&lt;cfoutput&gt;
	
&lt;cfloop array=&quot;#artists#&quot; index=&quot;artist&quot;&gt;
	&lt;h4&gt;#artist.firstName# #artist.lastname# #artist.id#&lt;/h4&gt;
	&lt;cfif artist.hasArt()&gt;
		&lt;ul&gt;
			&lt;cfloop array=&quot;#artist.getArt()#&quot; index=&quot;art&quot;&gt;
				&lt;li&gt;#art.name# #dollarFormat( art.price )#&lt;/li&gt;
			&lt;/cfloop&gt;
		&lt;/ul&gt;
	&lt;/cfif&gt;
&lt;/cfloop&gt;

&lt;/cfoutput&gt;

</pre>

Within the loop artist represents a single object. We can reference the data using the automatic getters in either the getProperty() format or, when, this.invokeImplicitAccessor is set to true in Application.cfc, by just the property. The same concept will be used for setters later.

The output will look like this:
<pre>
<h4>Jeff Baclawski</h4>
	
		<ul>
			
				<li>Bowl of Flowers $11,800.00</li>
			
				<li>60 Vibe $25,000.00</li>
			
				<li>Naked $30,000.00</li>
			
				<li>Sky $15,000.00</li>
			
				<li>Slices of Life $20,000.00</li>
			
				<li>sda $10.00</li>
			
		</ul>
</pre>



## More retrieving data with HQL

Hibernate Query Language (HQL) is a language similar to SQL for more complex data retrieval methods. In the following example we use like to get all records where first name begins with A. ormExecuteQuery will return an array of objects the same way as entityLoad.

<pre>
&lt;cfset artists = ormExecuteQuery( "FROM artist WHERE firstname like :firstname ORDER BY lastname", { firstname: "A%"})&gt;
</pre>

## Serializing as JSON

The serializeJSON function can serialize entities in JavaScript Object Notation (JSON) format.

<pre>
serializeJSON( artists )
</pre>
will produce:
<pre>
[{"firstname":"Jeff","id":4,"lastname":"Baclawski"}]
</pre>

# Working with Data

## Add Record

To add a record use the entityNew function and as a second argument pass in a structure (a map in other languages) of data. After adding data then call the entitySave function.

As we set up a relationship between artist and art when adding an artist we need to set their art as well. To do so we use the addXXX() function. In the example below this is achieved with the addArt function.

<pre>
transaction {

art = entityNew( "art", { name : "Painting of TV", price : 200, isSold : false } );
entitySave( art );

artist = entityNew( "artist", { firstname : "John", lastname : "Doe" } );
artist.addArt( art );

entitySave( artist );

}
</pre>

## Update Record

To update a record we need to retrieve a single object. To do so we use a different signature and the third argument is a boolean and needs to be true to retrieve a single record. Once we have a single record, values can be changed with the implicit setters. Any value changed will be saved to the database at the end of the transaction. If you want to retrieve a value saved in a transaction do so after the transaction has ended. To rollback any changes use transactionRollback().

<pre>
transaction {
artist = entityLoad( "artist", 100, true );
artist.firstname = "Fred";
}
</pre>

## Delete Record

To delete a record retrieve a single object and then pass it to the entityDelete function.

<pre>
transaction {
artist = entityLoad( "artist", 100, true );
entityDelete( artist );
}
</pre>

## Vital Tips

### ormReload()

After any changes to entity configuration (adding/removing a property, etc) it is necessary to call ormReload() for it to take affect. During development its common to place this in the onRequestStart function of Application.cfc. Depending on the size of application it can take 100 milliseconds to 5 seconds to run. 

### Null values
There are times when ColdFusion can not return an entity and instead the value assigned will be null. Consider the following code: 
<pre>
art = entityLoad( "art", 9999999, true );
</pre>

We are asking for one entity where the id is ridiculously high. Instead of erroring this will set the art variable to null. The isNull variable can test for this:
<pre>
art = entityLoad( "art", 9999999, true );
if ( isNull( art ) ) {
  // no art entity provided. write code to deal with it.
}
</pre>


### Dumping objects

When using writeDump ColdFusion will try and display the data from relationships. Sometimes this can be attempt to display way more data than desired. To prevent this provide some of the other attributes:

<pre>
writeDump( var=artists, top=2, showudfs=false )
</pre>
