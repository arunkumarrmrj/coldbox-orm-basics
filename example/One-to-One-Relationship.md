###### Application.cfc

	<cfset this.name = "MyBlog_one2one_link">
	<cfset this.ormenabled = "true">
	<cfset this.ormsettings.datasource = "ORM_EG">
	<cfset this.ormsettings.dbcreate = "dropcreate">


###### Employee.cfc

	<cfcomponent persistent="true" table="employee">
	<cfproperty name="EmployeeID" generator="native">
	<cfproperty name="PersonalObj" fieldtype="one-toone" linktable="EmployeePersonal" fkcolumn="EmployeeID" inversejoincolumn="PersonalID" cfc="personal" cascade="all" singularname="PersonalObj">
	<cfproperty name="LastName">
	<cfproperty name="FirstName">
	</cfcomponent>


###### Personal.cfc

	<cfcomponent persistent="true" table="personal">
	<cfproperty name="PersonalID" fieldtype="id" generator="native">
	<cfproperty name="EmployeeObj" fieldtype="one-toone" cfc="Employee" linktable="EmployeePersonal" fkcolumn="PersonalID" inverseJoinColumn="EmployeeID" cascade="all" inverse="true">
	<cfproperty name="SSN">
	<cfproperty name="FatherName">
	</cfcomponent>


###### index.cfm

	<cfset empObj = EntityNew("Employee")>
	<cfset empObj.setFirstName("James")>
	<cfset empObj.setLastName("Bond")>

	<cfset perObj = EntityNew("Personal")>
	<cfset perObj.setSSN("1-1-100")>
	<cfset perObj.setFatherName("Bond")>
	<cfset perObj.setEmployeeObj(empObj)>

	<cfset empObj.setPersonalObj(perObj)>
	<cfset EntitySave(empObj)>
	<cfset ormflush()>
