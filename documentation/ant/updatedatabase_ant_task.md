---
layout: default
title: Docs | Updatedatabase ant task 
subnav: subnav_documentation.md
---

# updateDatabase Ant Task 

Roll the database forward applying all change sets that have not been run.  

# Parameters

<table>
    <tr>
        <th>Attribute</th>
        <th>Description</th>
        <th>Required</th>
    </tr>
    <tr>
        <td>changeLogFile</td>
        <td>The change log file to run</td>
        <td>Yes</td>
    </tr>
    <tr>
        <td>contexts</td>
        <td>A comma separated list of <a href="../contexts.html">contexts</a> to execute. If not specified, all contexts are run.</td>
        <td>No</td>
    </tr>
    <tr>
        <td>outputFile</td>
        <td>If specified, Liquibase will save the update SQL statements to the specified file rather than executing them in the database.</td>
        <td>No</td>
    </tr>
    <tr>
        <td>outputEncoding</td>
        <td>The character encoding to use when writing SQL statements to output file.</td>
        <td>No; defaults to system encoding.</td>
    </tr>
    <tr>
        <td>promptOnNonLocalDatabase</td>
        <td>If set to true a dialog box with warn you if you attempt to run the Liquibase against a database that is not on localhost
        </td>
        <td>No; default is false.</td>
    </tr>
    <tr>
        <td>dropFirst</td>
        <td>If set to true, Liquibase will first drop all database objects owned by the connected user.</td>
        <td>No; default is false</td>
    </tr>
    <tr>
        <td>classpathref</td>
        <td>A reference to the classpath used to run the task with.</td>
        <td>No</td>
    </tr>
    <tr>
        <td>databaseref</td>
        <td>A reference to the database that Liquibase will connect to.</td>
        <td>Yes, unless a nested <code>&lt;database&gt;</code> element is present.</td>
    </tr>
    <tr>
        <td>driver</td>
        <td><b>Deprecated:</b> Name of the database driver to connect with.</td>
        <td>No</td>
    </tr>
    <tr>
        <td>url</td>
        <td><b>Deprecated:</b> Use <code>&lt;database&gt;</code>'s url attribute instead. The database URL</td>
        <td>No</td>
    </tr>
    <tr>
        <td>username</td>
        <td><b>Deprecated:</b>The database username to connect with</td>
        <td>No</td>
    </tr>
    <tr>
        <td>password</td>
        <td><b>Deprecated:</b>The password to use when connecting to the database</td>
        <td>No</td>
    </tr>
    <tr>
        <td>defaultSchemaName</td>
        <td><b>Deprecated:</b>Schema to use by default for managed database objects and Liquibase control tables</td>
        <td>No</td>
    </tr>
    <tr>
        <td>currentDateTimeFunction</td>
        <td><b>Deprecated:</b> Overrides current date time function used in SQL. Useful for unsupported databases</td>
        <td>No</td>
    </tr>
    <tr>
        <td>databaseChangeLogTableName</td>
        <td><b>Deprecated:</b> Overrides the name of the databasechangelog table to use</td>
        <td>No</td>
    </tr>
    <tr>
        <td>databaseChangeLogLockTableName</td>
        <td><b>Deprecated:</b> Overrides the name of the databasechangeloglock table to use</td>
        <td>No</td>
    </tr>
    <tr>
        <td>logLevel</td>
        <td><b>Deprecated:</b> Specifies one of the following logging levels: debug, info, warning, severe, off. The default level is info.</td>
        <td>No</td>
    </tr>
</table>

## Parameters Specified as Nested Elements

### classpath

The classpath used to run the task with. Optional. 

### database

Required unless a `databaseref` attribute is given. See [database data type](./index.html).

### changelogparameters

Optional. See [change log parameters](./index.html)

## Examples

{% highlight xml %}
<liquibase:updateDatabase changeLogFile="/path/to/changeLog.xml">
	<liquibase:database driver="${db.driver}" url="${db.url}" user="${db.user}" password="${db.pasword}"/>
</liquibase:updateDatabase>
{% endhighlight %}

A basic implementation of the update database task. Applies updates in the change log file to the database.

{% highlight xml %}
<database id="my-database" driver="${driver.classname}" url="${jdbc.url}" user="${username}" password="${password}"/>
<liquibase:updateDatabase databaseRef="my-database" changelogFile="/path/to/changeLog.xml"/>
{% endhighlight %}

Updates the referenced database with the change sets found in the given change log file.

{% highlight xml %}
<liquibase:updateDatabase changeLogFile="/path/to/changeLog.xml">
	<liquibase:database driver="${db.driver}" url="${db.url}" user="${db.user}" password="${db.pasword}"/>
	<liquibase:changeLogParameters>
		<liquibase:changeLogParameter name="property1" value="value1"/>
		<liquibase:changeLogParameter name="property2" value="value2"/>
		<propertyset>
			<propertyref prefix="mydb"/>
		</propertyset>
	</liquibase:changeLogParameters>
</liquibase:updateDatabase>
{% endhighlight %}

Updates the database with the change log file, substituting any change log parameters present with values defined in the nested elements.
