# ORM Examples


### One to Many Relationship with a LinkTable

###### Application.cfc

	<cfset this.name = "one-to-many">
	<cfset this.ormenabled = "true">
	<cfset this.ormsettings.datasource = "ORM">
	<cfset this.ormsettings.dbcreate = "dropcreate">


###### Category.cfc

	<cfcomponent persistent="true" table="category">
	<cfproperty name="CategoryID" fieldtype="id" generator="native">
	<cfproperty name="CategoryName">
	<cfproperty name="Description">
	<cfproperty name="products" fieldtype="one-to-many" linktable="CategoryProduct" type="array" cfc="product" inverse="true" cascade="all-delete-orphan" fkcolumn="categoryid" inversejoincolumn="productid" lazy="false" singularname="products">
	</cfcomponent>


###### Product.cfc

	<cfcomponent persistent="true" table="product">
	<cfproperty name="ProductID" fieldtype="id" generator="native">
	<cfproperty name="ProductName">
	<cfproperty name="CategoryID" fieldtype="many-toone" linktable="CategoryProduct" fkcolumn="ProductID" inversejoincolumn="CategoryID" cfc="category" singularname="Category">
	</cfcomponent>


###### index.cfm

	<!---Inserting one category and two products--->
	<cfset catObj = EntityNew("category")>
	<cfset catObj.setCategoryName("Software")>
	<cfset catObj.setDescription("ColdFusion Software")>

	<cfset prodObj1 = EntityNew("product")>
	<cfset prodObj1.setProductName("ColdFusion 9 Server")>
	<cfset prodObj1.setCategoryID(catObj)>

	<cfset prodObj2 = EntityNew("product")>
	<cfset prodObj2.setProductName("ColdFusion Builder")>
	<cfset prodObj2.setCategoryID(catObj)>

	<cfset catObj.addProducts(prodobj1)>
	<cfset catObj.addProducts(prodObj2)>

	<cfset EntitySave(catObj)>
	<cfset ormflush()>
