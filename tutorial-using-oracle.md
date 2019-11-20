---
layout: default
title: Tutorial Using Liquibase | Liquibase Docs
includeDaticalBox: true
---

<div>
<p>
This is a Liquibase tutorial that shows you how to manage your database objects using the Oracle database and some Oracle tools. It also shows how branching and merging is performed using Subversion. The tutorial incorporates several best practices. This makes it realistic for the needs of a software development shop. Of course your needs may be simpler. You may not need branching, you may not ever need to deliver a fresh install. In that case, you can just use the best practices and conventions that apply to your situation.
</p>

<p>
The scenario we will be using for the tutorial assumes we are developing an application for two customers named Solo and Duplex. Each customer has their own upgrade timing.
</p>
<div ><table >
	<tr >
		<th >Time </th><th > Solo          </th><th > Duplex         </th>
	</tr>
	<tr >
		<td >t1 </td><td > install r1.0    </td><td >                </td>
	</tr>
	<tr >
		<td >t2 </td><td > patch to r1.1   </td><td >                </td>
	</tr>
	<tr >
		<td >t3 </td><td > upgrade to r2.0 </td><td > install r2.0   </td>
	</tr>
</table></div>

<p>
During the lifecycle of an application, the development team may produce thousands of database changes. Periodically, we will want to consolidate these changes so that a fresh install can be done rapidly (also essential for the continuous integration environment), and so that we can ship a smaller set of changes to customers for upgrading.
</p>

<p>
The chosen strategy, is to clean up the changelog for every major release. This means that customers will have to upgrade to major releases separately. I.e. to upgrade from release 1.0 to release 3.0 they will also have to install 2.0.
A delivery for version X (this includes versions X.y) always consists of 3 parts:
</p>
<div ><table >
	<tr >
		<th >Script</th><th >Description</th><th >Create objects for version X </th><th > Upgrade from X-1</th><th >Apply latest changes to version X</th>
	</tr>
	<tr >
		<td >lb_install</td><td >Perform fresh install for version X</td><td >  X  </td><td > </td><td >  X  </td>
	</tr>
	<tr >
		<td >lb_upgrade_to_major</td><td >Upgrade from (X-1) to X </td><td > </td><td >  X  </td><td >  X  </td>
	</tr>
	<tr >
		<td >lb_update</td><td >Upgrade within same major version, eg 1.1 to 1.5</td><td >  </td><td > </td><td >  X  </td>
	</tr>
</table></div>

<p>
The directory structure we will build during this tutorial is shown below (assuming that we are delivering version 5.x). Instead of using a Subversion server, we will use a local repository in the <code>repo</code> directory.
</p>
<pre >D:\projects
    +-- lbdemo
          +-- repo           &lt;-- Subversion repository
          +-- trunk          &lt;-- Working directory
                +-- install  &lt;-- Installation changelogs
                +-- latest   &lt;-- Changelogs for replaceable objects (packages, triggers, views)
                +-- v004     &lt;-- Changelogs to upgrade from version 4.0
                +-- v005     &lt;-- Changelogs to upgrade from version 5.0</pre>

<p>
<strong>lb_install:</strong> A fresh install for version 5.0 will run:
</p>
<ul>
<li ><div > the changelogs in <code>install</code></div>
</li>
<li ><div > the changelogs in <code>latest</code> creating &#039;replaceable&#039; objects</div>
</li>
</ul>

<p>
<strong>lb_upgrade_to_major:</strong> An upgrade from version 4.x to 5.0 will run:
</p>
<ul>
<li ><div > the changelogs in <code>v004</code></div>
</li>
<li ><div > the changelogs in <code>v005</code></div>
</li>
</ul>

<p>
<strong>lb_update:</strong> An update from version 5.x will run:
</p>
<ul>
<li ><div > the changelogs in <code>v005</code></div>
</li>
</ul>

<p>
The scenario for this tutorial is illustrated in the diagram below. It will be handy to print this diagram for reference while you are going through the tutorial. Each of the blue boxes is a Subversion revision. The contents of a box is the commit message which describes the change.
</p>

<p>
<a href="custom_images/tutorial-overview.png"  title="tutorial-overview.png"><img src="custom_images/tutorial-overview.png"  alt="" width="800" /></a>
</p>

</div>

<h2><a>Step 1: Install the software</a></h2>

<h3><a>Install Oracle Database</a></h3>
<div >

<p>
If you don&#039;t already have an Oracle database available, then download Oracle XE from the <a href="http://www.oracle.com/technology"  title="http://www.oracle.com/technology"  rel="nofollow">Oracle Technology Network</a> and install it on your computer. The XE edition of the database is free to use for development and production purposes.
</p>

</div>

<h3><a>Install SQL Developer</a></h3>
<div >

<p>
There are many tools available to manage your database objects. If you haven&#039;t already got such a tool, then take a look at <acronym title="Structured Query Language">SQL</acronym> Developer. <acronym title="Structured Query Language">SQL</acronym> Developer is available as a free download at the <a href="http://www.oracle.com/technology"  title="http://www.oracle.com/technology"  rel="nofollow">Oracle Technology Network</a>. 
</p>

</div>

<h3><a>Install JDeveloper</a></h3>
<div >

<p>
Oracle JDeveloper is another free tool available from the <a href="http://www.oracle.com/technology"  title="http://www.oracle.com/technology"  rel="nofollow">Oracle Technology Network</a>. It has too many features to list here, we will only mention the <acronym title="Extensible Markup Language">XML</acronym> editor. This editor is schema aware, and makes it easy to author your changelogs.
</p>

<p>
After downloading and installing, start JDeveloper. If you are behind a proxy, configure the proxy using:
</p>

<p>
Tools &gt; Preferences &gt; Web Browser and Proxy
</p>

<p>
Next we will add the Liquibase <acronym title="XML (Extensible Markup Language) Schema Definition">XSD</acronym> so that the editor becomes aware of the Liquibase schema.
Choose Tools &gt; Preferences &gt; <acronym title="Extensible Markup Language">XML</acronym> Schemas. Select "add". Enter:
</p>
<div ><table >
	<tr >
		<th >Schema:</th><td > http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd </td>
	</tr>
	<tr >
		<th >Extension: </th><td >xml</td>
	</tr>
</table></div>

<p>
Press OK and now your JDeveloper is ready to edit your changelogs. There is no need to create an application or a project. Simply open the <acronym title="Extensible Markup Language">XML</acronym> file you want to edit.
</p>

</div>

<h3><a>Install TortoiseSVN</a></h3>
<div >

<p>
TortoiseSVN is a graphical front end to Subversion. It also has the capability to create a file based repository. We will use this feature for this tutorial. There is no need for you to have access to a Subversion server. Download TortoiseSVN from <a href="http://tortoisesvn.net/"  title="http://tortoisesvn.net/"  rel="nofollow">http://tortoisesvn.net/</a>.
Install it and reboot your PC.
</p>

</div>

<h3><a>Create directory structure and Subversion repository</a></h3>
<div >

<p>
Create the directory structure for this tutorial.
</p>
<pre >D:
mkdir D:\projects\lbdemo\repo</pre>

<p>
In Windows Explorer, right-click on directory repo and choose <strong>TortoiseSVN &gt; Create repository here</strong>. We have now created a Subversion repository.
</p>

<p>
Right-click on the directory repo again, and choose  <strong>TortoiseSVN &gt; Repo-browser</strong>. The browser displays in a window similar to Windows Explorer. Right-click on the repo directory and choose <strong>Create folder ..</strong>. Enter the name: <code>trunk</code>. You will then be prompted for a comment before you commit this change. Enter any comment.
</p>

<p>
Repeat this to create folders <code>branches</code> and <code>tags</code>.
</p>

<p>
You should end up with a repository structure that looks like this:
</p>
<pre >file:///D:/projects/lbdemo/repo
  +- branches
  +- tags
  +- trunk</pre>

<p>
Close the repository browser.
</p>

<p>
Return to Windows Explorer, navigate to directory D:\projects\lbdemo and right-click in the right panel. Choose <strong>SVN Checkout</strong>. In the panel that appears, enter:
</p>
<div ><table >
	<tr >
		<th ><acronym title="Uniform Resource Locator">URL</acronym> of repository: </th><td ><code>file:///D:/projects/lbdemo/repo/trunk</code></td>
	</tr>
	<tr >
		<th >Checkout directory: </th><td ><code>D:\projects\lbdemo\trunk</code></td>
	</tr>
</table></div>

<p>
Press OK. A confirmation window will display to create directory trunk. Select Yes. Press OK. In Windows Explorer, the trunk directory will be displayed with a green icon, indicating that it is a working copy with no changes to commit.
</p>

</div>

<h3><a>Install the JDBC Driver</a></h3>
<div >

<p>
Download the Oracle JDBC driver here: <a href="http://www.oracle.com/technology/software/tech/java/sqlj_jdbc/index.html"  title="http://www.oracle.com/technology/software/tech/java/sqlj_jdbc/index.html"  rel="nofollow">http://www.oracle.com/technology/software/tech/java/sqlj_jdbc/index.html</a>
Click on the link for 10g  Release 2 drivers, and choose <code>ojdbc14.jar</code>.
</p>

<p>
Save the jar in directory: <code>D:\projects\lbdemo</code>.
</p>

</div>

<h3><a>Install Liquibase</a></h3>
<div >

<p>
Install Liquibase following the instructions.
</p>

<p>
Add the directory containing the <code>Liquibase.bat</code> file to your PATH. <a href="http://redmondlab.net"  title="http://redmondlab.net"  rel="nofollow">Redmond Path</a> is a handy tool to edit your path; much easier than the standard facility in Windows. 
</p>

<p>
Test the installation by opening a DOS box in the <code>D:\projects\lbdemo\trunk</code> directory and entering the following command:
</p>
<pre >Liquibase --version</pre>

<p>
The result should be something like: <code>Liquibase Version: 1.9.0</code>
</p>

<p>
Next, create a file named <code>Liquibase.properties.template</code> in the <code>trunk</code> directory with the following contents:
</p>
<pre >#Liquibase.properties
driver: oracle.jdbc.OracleDriver
classpath: ../ojdbc14.jar
url: jdbc:oracle:thin:@localhost:1521:XE
username: tbd
password: tbd</pre>

<p>
Note that the classpath is relative to the current directory.
</p>

<p>
Copy this file to <code>Liquibase.properties</code> and edit the last 2 lines to look like this:
</p>
<pre >#Liquibase.properties
driver: oracle.jdbc.OracleDriver
classpath: ../ojdbc14.jar
url: jdbc:oracle:thin:@localhost:1521:XE
username: lb_dev
password: lb_dev</pre>

<p>
This properties file will save us from specifying these parameters on the command line. We will check the template file into subversion. The file with the real connection details we will keep locally.
</p>

<p>
Right-click on file <code>Liquibase.properties</code> and choose <strong>TortoiseSVN &gt; Add to ignore list &gt; Liquibase.properties</strong>
</p>

<p>
Right-click on file <code>Liquibase.properties.template</code> and choose <strong>TortoiseSVN &gt; Add</strong>
</p>

<p>
Right-click on directory <code>trunk</code> and choose <strong>TortoiseSVN &gt; commit</strong>.
</p>

</div>

<h2><a>Step 2: Create the initial database schemas</a></h2>
<div >

<p>
For our tutorial, we will create these database schemas:
</p>
<ul>
<li ><div > <strong>lb_dev</strong> - our development environment</div>
</li>
<li ><div > <strong>lb_test</strong> - our test environment</div>
</li>
<li ><div > <strong>lb_solo</strong> - the environment of customer Solo</div>
</li>
<li ><div > <strong>lb_duplex</strong> - the environment of customer Duplex</div>
</li>
</ul>

<p>
Open <acronym title="Structured Query Language">SQL</acronym> Developer and create a connection to your database using the "system" username.
</p>

<p>
File &gt; New &gt; Connections &gt; Database connection
</p>

<p>
Open this connection and run these commands:
</p>
<pre >drop user lb_dev cascade;
create user lb_dev identified by lb_dev;
grant connect, resource, create view to lb_dev;

drop user lb_test cascade;
create user lb_test identified by lb_test;
grant connect, resource, create view to lb_test;

drop user lb_solo cascade;
create user lb_solo identified by lb_solo;
grant connect, resource, create view to lb_solo;

drop user lb_duplex cascade;
create user lb_duplex identified by lb_duplex;
grant connect, resource, create view to lb_duplex;</pre>

<p>
In <acronym title="Structured Query Language">SQL</acronym> Developer, create a connection for each of these users. They will come in handy later.
</p>

</div>

<h2><a>Step 3: Create project directories and standard files</a></h2>
<div >

<p>
In this step we will be creating the following directory structure in the <code>trunk</code> directory:
</p>
<pre >trunk
  +-- install
  |     +-- cst      &lt;-- Contains FK constraints, one file per table
  |     +-- seq      &lt;-- Contains sequence definitions, one file per sequence
  |     +-- tab      &lt;-- Contains table definitions, one file per table
  +-- latest
  |     +-- pkb      &lt;-- Contains package bodies, one file per package
  |     +-- pks      &lt;-- Contains package specifications, one file per package
  |     +-- trg      &lt;-- Contains triggers, one file per trigger
  |     +-- vw       &lt;-- Contains views, one file per view
  +-- v000</pre>

<p>
We will create a separate install directory for the installation of non-replaceable objects (as opposed to the upgrades), and within the install directory a separate directory for each type of object.
</p>

<p>
Under latest, we have a directory for each type of "replaceable" database object. By "replaceable" we mean that the object can be updated by simply replacing it by a new version. The <acronym title="Structured Query Language">SQL</acronym> syntax for these objects starts with <code>create or replace</code>.
</p>

<p>
Paste these commands in a DOS box to create the directory structure in one go:
</p>
<pre >cd \projects\lbdemo\trunk
mkdir install
mkdir install\cst
mkdir install\seq
mkdir install\tab
mkdir latest
mkdir latest\pkb
mkdir latest\pks
mkdir latest\trg
mkdir latest\vw
mkdir v000</pre>

<p>
As we are starting from scratch (version 0), directory v000 will contain the changelogs to migrate from version 0.
</p>

<p>
We&#039;ll create a simple batch file to perform the upgrades:
</p>

<p>
<strong>trunk/lb_update.bat</strong>
</p>
<pre >@echo off
call Liquibase --changeLogFile=update.xml update</pre>

<p>
The file above refers to <code>update.xml</code>, which we will create next.
</p>

<p>
JDeveloper is a great tool for editing <acronym title="Extensible Markup Language">XML</acronym> files, but creating a new <acronym title="Extensible Markup Language">XML</acronym> file from scratch is a bit cumbersome. The following steps are the quickest way:
</p>
<ul>
<li ><div > Create an empty file in Windows Explorer (New &gt; TextDocument) and give it the correct filename</div>
</li>
<li ><div > Open this file in JDeveloper</div>
</li>
<li ><div > Copy and paste the file contents from this tutorial into JDeveloper</div>
</li>
</ul>

<p>
<strong>trunk/update.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;include file=&quot;v000/master.xml&quot; /&gt;
&lt;/databaseChangeLog&gt;
</pre>

<p>
Create the file that will later contain the includes of each changeLog in order of applicability.
</p>

<p>
<strong>trunk/v000/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 0 --&gt;
        &lt;sqlCheck expectedResult=&quot;0&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;

&lt;/databaseChangeLog&gt;</pre>

<p>
You may be wondering why this file contains a preCondition. Doesn&#039;t Liquibase already check which changes have been run? Well, if we perform a fresh install of version 5, then the database will not contain the list of changes before version 5. And we definitely don&#039;t want to run these changes by accident. So we define an explicit check in the form of a preCondition.
</p>

<p>
Commit the current version to Subversion. Right-click on directory trunk and select <strong>SVN Commit</strong>
</p>

<p>
Enter a message like "Initial version", select all files/directories and press OK. Now the files/directories will also be displayed with a green icon in Windows explorer.
</p>

</div>

<h2><a>Step 4: Create database objects</a></h2>
<div >

<p>
Each change (including initial creations) are related to an issue number: a bug or a project task. Our first task (which happens to have number 73) is to create 2 tables.
</p>

<p>
The file structure which we will be describing has a certain structure which is easier to see if we visualize it. The white boxes are the directories. The blue boxes are the files within the directories. The arrows indicate that one file calls or includes another file.
</p>

<p>
<a href="custom_images/tutorial-files.png"  title="custom_images/tutorial-files.png"><img src="custom_images/tutorial-files.png"  alt="" width="600" /></a>
</p>

</div>

<h3><a>Change 73</a></h3>
<div >

<p>
The description of this task is:
</p>
<ul>
<li ><div > Create table <code>departments</code>, the primary key is populated using a sequence and a trigger</div>
</li>
<li ><div > Create table <code>employees</code> with a foreign key to departments</div>
</li>
</ul>

<p>
We will create one changefile for these changes. It will consist of these changes:
</p>
<ul>
<li ><div > createTable departments</div>
</li>
<li ><div > createSequence</div>
</li>
<li ><div > include of a separate file for the trigger. This file is located in directory <code>latest/trg</code></div>
</li>
<li ><div > createTable employees</div>
</li>
</ul>

<p>
Create the following files:
</p>

<p>
<strong>trunk/v000/2009-10-15-73.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;createTable tableName=&quot;departments&quot;
                     remarks=&quot;The departments of this company. Does not include geographical divisions.&quot;&gt;
            &lt;column name=&quot;id&quot; type=&quot;number(4,0)&quot;&gt;
                &lt;constraints nullable=&quot;false&quot; primaryKey=&quot;true&quot;
                             primaryKeyName=&quot;DPT_PK&quot;/&gt;
            &lt;/column&gt;
            &lt;column name=&quot;dname&quot; type=&quot;varchar2(14)&quot;
                    remarks=&quot;The official department name as registered on the internal website.&quot;/&gt;
        &lt;/createTable&gt;
        &lt;addUniqueConstraint constraintName=&quot;departments_uk1&quot;
                             columnNames=&quot;dname&quot; tableName=&quot;departments&quot;/&gt;
        &lt;createSequence sequenceName=&quot;departments_seq&quot;/&gt;
    &lt;/changeSet&gt;
    
    &lt;include file=&quot;latest/trg/departments_bi.xml&quot;/&gt;
    
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;2&quot;&gt;
        &lt;createTable tableName=&quot;employees&quot;&gt;
            &lt;column name=&quot;id&quot; type=&quot;number(4,0)&quot;&gt;
                &lt;constraints nullable=&quot;false&quot; primaryKey=&quot;true&quot;
                             primaryKeyName=&quot;EMP_PK&quot;/&gt;
            &lt;/column&gt;
            &lt;column name=&quot;ename&quot; type=&quot;varchar2(14)&quot;
                    remarks=&quot;The first and last name&quot;/&gt;
            &lt;column name=&quot;salary&quot; type=&quot;number(6,2)&quot;
                    remarks=&quot;The full remuneration&quot;/&gt;
            &lt;column name=&quot;dpt_id&quot; type=&quot;number(4,0)&quot;/&gt;
        &lt;/createTable&gt;
        &lt;addForeignKeyConstraint baseColumnNames=&quot;dpt_id&quot;
                                 baseTableName=&quot;employees&quot;
                                 constraintName=&quot;emp_dpt_fk&quot;
                                 referencedColumnNames=&quot;id&quot;
                                 referencedTableName=&quot;departments&quot;/&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Note that the table and column remarks will be applied as table and column comments in the database.
</p>

<p>
<strong>trunk/latest/trg/departments_bi.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot; runOnChange=&quot;true&quot;&gt;
        &lt;createProcedure&gt;
create trigger departments_bi before insert on departments
for each row
when (new.id is null)
begin
 select departments_seq.nextval into :new.id from dual;
end;
        &lt;/createProcedure&gt;
        &lt;rollback&gt;
            drop trigger departments_bi
        &lt;/rollback&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Note the runOnChange="true" attribute. This ensures that we can make future changes in this file and these changes will be applied automatically. Note also that in the file above, we have provided an explicit rollback statement to undo this change. Liquibase can automatically generate rollback statements for many commands, but not for <acronym title="Structured Query Language">SQL</acronym> blocks in which anything may happen.
</p>

<p>
Update the master.xml to include the change:
</p>

<p>
<strong>trunk/v000/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 0 --&gt;
        &lt;sqlCheck expectedResult=&quot;0&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v000/2009-10-15-73.xml&quot; /&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Now we are ready to test the changes by applying them to the database. Run the batch file we created before:
</p>
<pre >D:\projects\lbdemo\trunk&gt; lb_update</pre>

<p>
If all goes well, you will see a confirmation message: Migration successful.
</p>

<p>
Now examine the contents of table DATABASECHANGELOG using <acronym title="Structured Query Language">SQL</acronym>*Developer. You will see 3 rows, corresponding to the 3 changeSets we applied. If the change was not exactly what you wanted, you can roll it back. To view the <acronym title="Structured Query Language">SQL</acronym> that will roll back the changes:
</p>
<pre >Liquibase --changeLogFile=update.xml rollbackCountSQL 3</pre>

<p>
To actually perform the rollback:
</p>
<pre >Liquibase --changeLogFile=update.xml rollbackCount 3</pre>

<p>
After you have completed and tested your changes, commit them to Subversion with this comment: "73: Create tables departments &amp; employees". The current state is represented by the first box on the diagram at the start of this tutorial.
</p>

<p>
This task is now complete, we will continue to the next task.
</p>

</div>

<h3><a>Change 59</a></h3>
<div >

<p>
This description of this task is:
</p>
<ul>
<li ><div > Create package <code>departments_pck</code></div>
</li>
<li ><div > Create view <code>departments_vw</code></div>
</li>
</ul>

<p>
Both objects are of the "replaceable" type, i.e. a new version can simply replace an older version. The files will therefore go into the <code>latest</code> directory. Create/update the following files:
</p>

<p>
<strong>trunk/v000/master.xml</strong> (add the new change to the end of the file)
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 0 --&gt;
        &lt;sqlCheck expectedResult=&quot;0&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v000/2009-10-15-73.xml&quot; /&gt;
    &lt;include file=&quot;v000/2009-10-15-59.xml&quot; /&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/v000/2009-10-15-59.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;include file=&quot;latest/pks/departments_pck.xml&quot;/&gt;
    &lt;include file=&quot;latest/pkb/departments_pck.xml&quot;/&gt;
    &lt;include file=&quot;latest/vw/departments_vw.xml&quot;/&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/latest/pks/departments_pck.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot; runOnChange=&quot;true&quot;&gt;
        &lt;createProcedure&gt;
create or replace package departments_pck as
  function  upname(name in varchar2) return varchar2;
end departments_pck;
        &lt;/createProcedure&gt;
        &lt;rollback&gt;
            drop package departments_pck
        &lt;/rollback&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/latest/pkb/departments_pck.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot; runOnChange=&quot;true&quot;&gt;
        &lt;createProcedure&gt;
create or replace package body departments_pck as
  function upname(name in varchar2) return varchar2 is
  begin
    return upper(name);
  end;
end departments_pck;
        &lt;/createProcedure&gt;
        &lt;rollback&gt;
            drop package body departments_pck
        &lt;/rollback&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/latest/vw/departments_vw.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot; runOnChange=&quot;true&quot;&gt;
        &lt;createView viewName=&quot;departments_vw&quot;&gt;
            select id, dname
            from departments
        &lt;/createView&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Now we are ready to test the changes by applying them to the database. Run the batch file we created before:
</p>
<pre >D:\projects\lbdemo\trunk&gt; lb_update</pre>

<p>
If all goes well, you will see a confirmation message: Migration successful.
</p>

<p>
Now examine the contents of table DATABASECHANGELOG using <acronym title="Structured Query Language">SQL</acronym>*Developer. You will see 3 new rows, corresponding to the 3 changeSets we applied. Try rolling back the changes, to test the rollback statements we coded:
</p>
<pre >Liquibase --changeLogFile=update.xml rollbackCount 3</pre>

<p>
After you have completed and tested your changes, commit them to Subversion with comment:"59: create departments_pck and departments_vw".
</p>

<p>
We could continue in this manner for the remainder of the project. At some point however, the number of changes may become very large and we may want to define a new starting point. This is the topic of the next section.
</p>

</div>

<h2><a>Step 5: Branch major release</a></h2>
<div >

<p>
We are ready to branch for release 1.x. This will allow us to perform a fresh install of version 1.x, without applying the many changes from 0.0 to 1.0. Of course, for installations running version 0.x, we also provide the incremental migration possibility.
</p>

</div>

<h3><a>Step 5.1: Finalize this version</a></h3>
<div >

<p>
The last change to version 0 has been made. After all these changes have been applied, we are effectively at major version 1. The major version is recorded in table databasechangelog. Modify this file to record this version:
</p>

<p>
<strong>trunk/v000/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!--These changes should only be run against a schema with major version 0--&gt;
        &lt;sqlCheck expectedResult=&quot;0&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v000/2009-10-15-73.xml&quot;/&gt;
    &lt;include file=&quot;v000/2009-10-15-59.xml&quot;/&gt;
    
    &lt;!-- Do not include any more changes in this file --&gt;
    &lt;changeSet author=&quot;MajorVersion&quot; id=&quot;1&quot; /&gt;

&lt;/databaseChangeLog&gt;</pre>

<p>
Run the update command to apply the changeset with the version number:
</p>
<pre >lb_update</pre>

</div>

<h3><a>Step 5.2: Create fresh-install scripts</a></h3>
<div >

<p>
Liquibase stores sets of changes in a <strong>changelog</strong>. Although we can create the changelog for the initial objects by hand, it would be nice to be able to generate them from an existing database. The Liquibase generateChangeLog command does export tables to a changelog file, but it has some significant shortcomings:
</p>
<ul>
<li ><div > Everything is dumped into one file. This does not provide much overview in your version control system.</div>
</li>
<li ><div > It does not export all objects. E.g. triggers and packages are missing.</div>
</li>
</ul>

<p>
So there is room here for a new utility (watch this space!).
</p>

<p>
For now, we will create the installation changelogs manually. The organization of the changelogs is as follows:
</p>
<ul>
<li ><div > The "replaceable" objects are stored in the <code>latest</code> directory, as we have seen before.</div>
</li>
<li ><div > Other objects are stored in the <code>install</code> directory.</div>
</li>
</ul>

<p>
Our (manual) utility will create these files:
</p>

<p>
<strong>trunk/install/tab/departments.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;createTable tableName=&quot;departments&quot;
                     remarks=&quot;The departments of this company. Does not include geographical divisions.&quot;&gt;
            &lt;column name=&quot;id&quot; type=&quot;number(4,0)&quot;&gt;
                &lt;constraints nullable=&quot;false&quot; primaryKey=&quot;true&quot;
                             primaryKeyName=&quot;DPT_PK&quot;/&gt;
            &lt;/column&gt;
            &lt;column name=&quot;dname&quot; type=&quot;varchar2(14)&quot;
                    remarks=&quot;The official department name as registered on the internal website.&quot;/&gt;
        &lt;/createTable&gt;
        &lt;addUniqueConstraint constraintName=&quot;departments_uk1&quot; columnNames=&quot;dname&quot;
                             tableName=&quot;departments&quot; /&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/install/tab/employees.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;createTable tableName=&quot;employees&quot;&gt;
            &lt;column name=&quot;id&quot; type=&quot;number(4,0)&quot;&gt;
                &lt;constraints nullable=&quot;false&quot; primaryKey=&quot;true&quot;
                             primaryKeyName=&quot;EMP_PK&quot;/&gt;
            &lt;/column&gt;
            &lt;column name=&quot;ename&quot; type=&quot;varchar2(14)&quot;
                    remarks=&quot;The first and last name&quot;/&gt;
            &lt;column name=&quot;salary&quot; type=&quot;number(6,2)&quot;
                    remarks=&quot;The full remuneration&quot;/&gt;
            &lt;column name=&quot;dpt_id&quot; type=&quot;number(4,0)&quot;/&gt;
        &lt;/createTable&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/install/seq/departments_seq.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;createSequence sequenceName=&quot;departments_seq&quot;/&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
<strong>trunk/install/cst/employees.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;addForeignKeyConstraint baseColumnNames=&quot;dpt_id&quot;
                                 baseTableName=&quot;employees&quot;
                                 constraintName=&quot;emp_dpt_fk&quot;
                                 referencedColumnNames=&quot;id&quot;
                                 referencedTableName=&quot;departments&quot;/&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

</div>

<h3><a>Step 5.3: Reorganize changelog directories and scripts</a></h3>
<div >

<p>
Create a new directory for the changelogs from 1.0:
</p>
<pre >mkdir v001</pre>

<p>
Create a new master.xml in this directory:
</p>

<p>
<strong>trunk/v001/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 1 --&gt;
        &lt;sqlCheck expectedResult=&quot;1&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Edit the update file:
</p>

<p>
<strong>trunk/update.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;include file=&quot;v001/master.xml&quot; /&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Create the install script for fresh installs of version 1.x:
</p>

<p>
<strong>trunk/lb_install.bat</strong>
</p>
<pre >@echo off
call Liquibase --changeLogFile=install.xml update
call Liquibase --changeLogFile=update.xml  update</pre>

<p>
As you can see above, after the installation of 1.x, we run the updates from 1.x to the latest version. You may wonder: why not include the update at the end of the install.xml file as: <code>&lt;include file="v001/master.xml" /&gt;</code> ? That would be equally valid. However, for some reason, if we do that then the precondition in master.xml fails.
</p>

<p>
Create the <code>install.xml</code>. This file does the fresh install of objects, records the MajorVersion number and includes new changes from version v001 (see last 2 lines in script below):
</p>

<p>
<strong> trunk/install.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;includeAll path=&quot;install/tab/&quot; /&gt;
    &lt;includeAll path=&quot;install/seq&quot; /&gt;
    &lt;includeAll path=&quot;install/cst&quot; /&gt;
    &lt;includeAll path=&quot;latest/pks&quot; /&gt;
    &lt;includeAll path=&quot;latest/vw&quot; /&gt;
    &lt;includeAll path=&quot;latest/pkb&quot; /&gt;
    &lt;includeAll path=&quot;latest/trg&quot; /&gt;

    &lt;changeSet author=&quot;MajorVersion&quot; id=&quot;1&quot; /&gt;

&lt;/databaseChangeLog&gt;</pre>

</div>

<h3><a>Step 5.4: Test the Fresh Install</a></h3>
<div >

<p>
Modify Liquibase.properties and change the username and password:
</p>
<pre >username: lb_test
password: lb_test</pre>

<p>
Perform the fresh install in schema lb_test:
</p>
<pre >lb_install</pre>

<p>
This should run successfully.
</p>

<p>
Modify Liquibase.properties and change the username and password back to:
</p>
<pre >username: lb_dev
password: lb_dev</pre>

<p>
To verify that the fresh install in lb_test is identical to the incrementally built schema in lb_dev we can use the Liquibase diff command. Enter the following command on the command-line:
</p>
<pre >Liquibase diff --referenceUrl=jdbc:oracle:thin:@localhost:1521:XE --referenceUsername=lb_test --referencePassword=lb_test</pre>

<p>
This command will give the output in text format and should indicate that there are no differences:
</p>
<pre >Diff Results:
Base Database: LB_TEST jdbc:oracle:thin:@localhost:1521:XE
Target Database: LB_DEV jdbc:oracle:thin:@localhost:1521:XE
Product Name: EQUAL
Product Version: EQUAL
Missing Tables: NONE
Unexpected Tables: NONE
Missing Views: NONE
Unexpected Views: NONE
Missing Columns: NONE
Unexpected Columns: NONE
Changed Columns: NONE
Missing Foreign Keys: NONE
Unexpected Foreign Keys: NONE
Missing Primary Keys: NONE
Unexpected Primary Keys: NONE
Missing Unique Constraints: NONE
Unexpected Unique Constraints: NONE
Missing Indexes: NONE
Unexpected Indexes: NONE
Missing Sequences: NONE
Unexpected Sequences: NONE</pre>

<p>
Liquibase can also produce output in changelog format. Look up the <code>diffChangeLog</code> command in the manual.
</p>

<p>
Release 1.x is technically correct now, ready for branching.
</p>

<p>
Commit the current version to Subversion. Right-click on directory trunk and select <strong>SVN Commit</strong>. Include all the newly created files.
</p>

<p>
Enter a message like "Finalize 1.x", select all files and press OK. Now the files will also be displayed with a green icon in Windows explorer.
</p>

</div>

<h3><a>Step 5.5: Branch version 1.x</a></h3>
<div >

<p>
Right-click on the <code>trunk</code> and choose <strong>TortoiseSVN &gt; Repo-browser</strong>. Select <code>trunk</code> in the left pane. By default we are selecting the HEAD, which is correct. But if someone else commited a change on the trunk in the meantime, you would want to select the revision in which you finalized 1.x.
</p>

<p>
Right-click on <code>trunk</code> and choose <code>copy to ..</code>.
</p>

<p>
Fill in:
</p>
<div ><table >
	<tr >
		<th >New Name:</th><td > file:///D:/projects/lbdemo/repo/branches/1.x</td>
	</tr>
</table></div>

<p>
Enter log message "Branched 1.x" and press OK.
</p>

<p>
In the left pane under branches you will now see a folder named 1.x. Right-click on <code>branches/1.x</code> and choose <strong>Checkout</strong>.
</p>
<div ><table >
	<tr >
		<th ><acronym title="Uniform Resource Locator">URL</acronym> of repository:   </th><td > file:///D:/projects/lbdemo/repo/branches/1.x     </td>
	</tr>
	<tr >
		<th >Checkout directory:  </th><td > D:\projects\lbdemo\branch_1.x  </td>
	</tr>
</table></div>

<p>
The new directory will be created, containing the files in this branch. The system test can start, to verify that this version is fit to ship.
</p>

<p>
Let&#039;s do a fresh install from this directory into the lb_test schema.
</p>

<p>
Navigate to directory <code>branch_1.x</code> and copy file <code>Liquibase.properties.template</code> to <code>Liquibase.properties</code>. Change the connection details to:
</p>
<pre >username: lb_test
password: lb_test</pre>

<p>
Re-create the lb_test schema again using SQLDeveloper. Connect as system and run:
</p>
<pre >drop user lb_test cascade;
create user lb_test identified by lb_test;
grant connect, resource, create view to lb_test;</pre>

<p>
Perform the fresh install from the directory <code>branch_1.x</code>:
</p>
<pre >cd \projects\lbdemo\branch_1.x
lb_install</pre>

</div>

<h2><a>Step 6: Create test data</a></h2>
<div >

<p>
In order to test data migration functionality, let&#039;s insert some test data. In a <acronym title="Structured Query Language">SQL</acronym> Developer session, run this script:
</p>
<pre >insert into departments (id, dname) values (1, &#039;HQ&#039;);
insert into departments (id, dname) values (2, &#039;Sales&#039;);
insert into employees (id, ename, salary, dpt_id) values (1, &#039;King&#039;, 1200, 1);
insert into employees (id, ename, salary, dpt_id) values (2, &#039;Smith&#039;, 1000, 2);
commit;</pre>

<p>
Run this script in the <strong>lb_dev</strong> schema and in the <strong>lb_test</strong> schema. We won&#039;t create Liquibase scripts for this data, and we won&#039;t store the scripts in Subversion. The art of maintaining setup data and test data is worth a separate tutorial, so we won&#039;t cover it here. If you are curious, look up the <strong>insert data</strong> and <strong>load data</strong> functionality in the Liquibase manual.
</p>

</div>

<h2><a>Step 7: Fix bug 102</a></h2>
<div >

<p>
The system test on version 1.x has revealed a bug and it needs to be fixed before we can ship version 1.0. We fix bugs on the trunk, and then backport them to the relevant branches.
</p>

<p>
This change has been registered in our issue tracker with number 102. The change description is:
</p>
<ul>
<li ><div > replace column employees.salary by 2 new columns: fixed_salary and bonus (both NOT-NULL)</div>
</li>
<li ><div > The value of bonus should be salary *0.1</div>
</li>
<li ><div > The value of fixed_salary should be salary * 0.9</div>
</li>
</ul>

<p>
Remember that our changes are now taking us from version 1.x, so the changelogs go in directory v001. Create this changelog:
</p>

<p>
<strong> trunk/v001/2009-10-16-102.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;addColumn tableName=&quot;employees&quot;&gt;
            &lt;column name=&quot;fixed_salary&quot; type=&quot;number(6,2)&quot;
                    remarks=&quot;Monthly gross salary&quot;/&gt;
            &lt;column name=&quot;bonus&quot; type=&quot;number(6,2)&quot;
                    remarks=&quot;On-target monthly bonus&quot;/&gt;
        &lt;/addColumn&gt;
        &lt;update tableName=&quot;employees&quot;&gt;
            &lt;column name=&quot;fixed_salary&quot; valueNumeric=&quot;salary * 0.9&quot;/&gt;
            &lt;column name=&quot;bonus&quot; valueNumeric=&quot;salary * 0.1&quot;/&gt;
        &lt;/update&gt;
        &lt;addNotNullConstraint tableName=&quot;employees&quot; columnName=&quot;fixed_salary&quot; /&gt;
        &lt;addNotNullConstraint tableName=&quot;employees&quot; columnName=&quot;bonus&quot; /&gt;
        &lt;dropColumn tableName=&quot;employees&quot; columnName=&quot;salary&quot; /&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Update <code>master.xml</code> to include this file:
</p>

<p>
<strong>trunk/v001/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 1 --&gt;
        &lt;sqlCheck expectedResult=&quot;1&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v001/2009-10-16-102.xml&quot;/&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Run the update command:
</p>
<pre >lb_update</pre>

<p>
Note that this change cannot be rolled back automatically. If we wanted to be able to roll this back, we would have had to specify the rollback logic.
</p>

<p>
Use <acronym title="Structured Query Language">SQL</acronym> Developer to check that the changes have successfully been applied.
</p>
<pre >DESC employees

ame                 Null     Type
-------------------- -------- ------------
ID                   NOT NULL NUMBER(4)
ENAME                         VARCHAR2(14)
DPT_ID                        NUMBER(4)
FIXED_SALARY         NOT NULL NUMBER(6,2)
BONUS                NOT NULL NUMBER(6,2)

5 rows selected

SELECT * FROM employees;

ID   ENAME     DPT_ID    FIXED_SALARY   BONUS
---- --------- --------- -------------- -----
1    King      1         1080           120  
2    Smith     2         900            100  

2 rows selected</pre>

<p>
Commit these changes using TortoiseSVN. Enter log message: "102: Replace salary by fixed_salary and bonus".
</p>

</div>

<h2><a>Step 8: Backport bug 102 to branch 1.x</a></h2>

<h4><a>Perform the merge</a></h4>
<div >

<p>
Right-click on directory <code>branch_1.x</code> and choose <strong>TortoiseSVN &gt; Merge</strong>.
</p>
<div ><table >
	<tr >
		<th >Merge type: </th><td > Merge a range of revisions </td>
	</tr>
</table></div>

<p>
Press Next
</p>
<div ><table >
	<tr >
		<th ><acronym title="Uniform Resource Locator">URL</acronym> to merge from: </th><td > file:///D:/projects/lbdemo/repo/trunk </td>
	</tr>
	<tr >
		<th >Revision range to merge: </th><td > Use the [Show log] button to display the list of revisions. Select the change for bug 102. </td>
	</tr>
</table></div>

<p>
Press Next
</p>

<p>
Press Merge
</p>

<p>
Press OK to close the results window.
</p>

<p>
In Windows Explorer, you will see that <code>2009-10-16-102.xml</code> has been added to the v001 directory, and that <code>v001/master.xml</code> has been updated. Right-click on file <code>master.xml</code> and choose <strong>TortoiseSVN &gt; Diff</strong>. You will see that only the one line has been added to master.xml.
</p>

<p>
Commit branch_1.x with comment "102: backport".
</p>

</div>

<h4><a>Test the results</a></h4>
<div >

<p>
Run the update command:
</p>
<pre >cd D:\projects\lbdemo\branch_1.x
lb_update</pre>

<p>
Use <acronym title="Structured Query Language">SQL</acronym> Developer to confirm that the employees in schema lb_test now have a fixed salary and a bonus.
</p>

</div>

<h4><a>Tag this revision as 1.0</a></h4>
<div >

<p>
The acceptance test has now been completed, so we can label this version as our "1.0" release.
Right-click on <code>branch_1.x</code> and choose <strong>TortoiseSVN &gt; Branch/Tag</strong>. Fill out the dialogue as shown below:
</p>
<div ><table >
	<tr >
		<th >From WC at <acronym title="Uniform Resource Locator">URL</acronym>:                     </th><td > file:///D:/projects/lbdemo/repo/branches/1.x     </td>
	</tr>
	<tr >
		<th >To <acronym title="Uniform Resource Locator">URL</acronym>:                             </th><td > file:///D:/projects/lbdemo/repo/tags/1.0  </td>
	</tr>
	<tr >
		<th >Create copy in the repository from: </th><td > (leave as default)        </td>
	</tr>
	<tr >
		<th >Log message:                        </th><td > Tagged as 1.0                           </td>
	</tr>
</table></div>

<p>
Press OK and close the repository browser.
</p>

<p>
We can now deliver version 1.0 to the customer.
</p>

</div>

<h2><a>Step 9: Install 1.0 for Solo</a></h2>
<div >

<p>
We can now deliver version 1.0 to our customer Solo. Create directory <code>D:\projects\lbdemo\solo</code>. Right-click on this directory and choose TortoiseSVN &gt; Export. Fill out the dialogue as shown below:
</p>
<div ><table >
	<tr >
		<th ><acronym title="Uniform Resource Locator">URL</acronym> of repository:                  </th><td > file:///D:/projects/lbdemo/repo/tags/1.0  </td>
	</tr>
	<tr >
		<th >Export directory:                   </th><td > D:\projects\lbdemo\solo         </td>
	</tr>
</table></div>

<p>
Leave the rest as default and press OK. The files will be exported to the solo directory.
</p>

<p>
Copy file <code>Liquibase.properties.template</code> to <code>Liquibase.properties</code> and change the username and password to <code>lb_solo</code>.
</p>

<p>
Apply the changeLog for the initial installation and the patches of version 1.0 to the empty schema of Solo:
</p>
<pre >cd D:\projects\lbdemo\solo
lb_install</pre>

<p>
Twice, you should receive the confirmation "Migration successful", and Solo is running on 1.0.
</p>

</div>

<h2><a>Step 10: Further development on the trunk (change 105)</a></h2>
<div >

<p>
The specifications for this change are:
</p>
<ul>
<li ><div > add column mgr_id to the departments table. To be populated by the mgr_id of King.</div>
</li>
<li ><div > add a foreign key: departments.mgr_id -> employees.id</div>
</li>
</ul>

<p>
Create the changelog:
</p>

<p>
<strong>trunk/v001/2009-10-16-105.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;addColumn tableName=&quot;departments&quot;&gt;
            &lt;column name=&quot;mgr_id&quot; type=&quot;number(4,0)&quot;/&gt;
        &lt;/addColumn&gt;
        &lt;sql&gt;update departments
             set mgr_id = (select id
                           from employees
                           where ename=&#039;King&#039;);
        &lt;/sql&gt;
        &lt;addForeignKeyConstraint baseColumnNames=&quot;mgr_id&quot;
                                 baseTableName=&quot;departments&quot;
                                 constraintName=&quot;dpt_emp_fk1&quot;
                                 referencedColumnNames=&quot;id&quot;
                                 referencedTableName=&quot;employees&quot;/&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Update <code>master.xml</code> to include this file:
</p>

<p>
<strong>trunk/v001/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 1 --&gt;
        &lt;sqlCheck expectedResult=&quot;1&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v001/2009-10-16-102.xml&quot;/&gt;
    &lt;include file=&quot;v001/2009-10-16-105.xml&quot;/&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Run the update command:
</p>
<pre >lb_update</pre>

<p>
Use <acronym title="Structured Query Language">SQL</acronym> Developer to check that the changes have been applied correctly.
</p>

<p>
Commit these changes using TortoiseSVN, with comment "105: Add mgr_id and foreign key".
</p>

</div>

<h2><a>Step 11: Branch major release 2.x</a></h2>
<div >

<p>
This is the same as step 5. Repeat step 5 , replacing 1.x by 2.x.
</p>

<p>
However, there is a difference. Customers running 1.x need to be able to upgrade to 2.x. We did not cover that aspect previously. If a 1.x customer were simply to run lb_update, then it would fail on the precondition in the master.xml:
</p>
<pre >    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 2 --&gt;
        &lt;sqlCheck expectedResult=&quot;2&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;</pre>

<p>
So how do we ensure that all the changes between "finalize 1.x" and "finalize 2.x" have been applied before applying the 2.x changes? If you look at the diagram at the beginning of this tutorial, you will see that we are talking about change 105. Change 105 was never delivered in the 1.x branch and it is not contained in the v002/master.xml.
</p>

<p>
The answer is, we create 2 new files as part of step 5.3:
</p>

<p>
<strong>trunk/lb_upgrade_to_major.bat</strong>
</p>
<pre >@echo off
call Liquibase --changeLogFile=upgrade_to_major.xml update
call Liquibase --changeLogFile=update.xml update</pre>

<p>
<strong>trunk/upgrade_to_major.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;include file=&quot;v001/master.xml&quot; /&gt;
&lt;/databaseChangeLog&gt;</pre>

</div>

<h2><a>Step 12: Change 114, rename column</a></h2>
<div >

<p>
Testing on branch 2.x revealed a bug that we want fixed on branch 1.x as well. Column employees.ename needs to be renamed to full_name. Refer to the diagram at the start of this tutorial for a visual overview of this step. By now you should know the routine.
</p>

<p>
Create the changelog:
</p>

<p>
<strong>trunk/v002/2009-10-18-114.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;renameColumn newColumnName=&quot;full_name&quot; oldColumnName=&quot;ename&quot; tableName=&quot;employees&quot;/&gt;
    &lt;/changeSet&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Update <code>master.xml</code> to include this file:
</p>

<p>
<strong>trunk/v002/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 2 --&gt;
        &lt;sqlCheck expectedResult=&quot;2&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v002/2009-10-18-114.xml&quot;/&gt;
&lt;/databaseChangeLog&gt;</pre>

<p>
Run the update command:
</p>
<pre >lb_update</pre>

<p>
Use <acronym title="Structured Query Language">SQL</acronym> Developer to check that the changes have been applied correctly.
</p>

<p>
Commit these changes using TortoiseSVN, with comment "114: Renamed employee.ename to full_name".
</p>

</div>

<h2><a>Step 13: Backport change 114 to branch 1.x</a></h2>
<div >

<p>
This is the same as step 8 with a major exception. On the trunk, change 114 is recorded in directory v002 and in file v002/master.xml
</p>

<p>
Branch 1.x does not have a directory v002. We will create this directory to contain the changelog. However, note that we will <strong>not</strong> create a file named <code>v002/master.xml</code>. There should only be one master.xml on a branch, as it contains the order of the changes.
</p>

<p>
Change 114 will be referenced in <code>v001/master.xml</code> as follows:
</p>

<p>
<strong>branch_1.x/v001/master.xml</strong>
</p>
<pre >&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot; standalone=&quot;no&quot;?&gt;
&lt;databaseChangeLog xmlns=&quot;http://www.liquibase.org/xml/ns/dbchangelog&quot;
                   xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot;
                   xsi:schemaLocation=&quot;http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd&quot;&gt;
    &lt;preConditions&gt;
        &lt;!-- These changes should only be run against a schema with major version 1 --&gt;
        &lt;sqlCheck expectedResult=&quot;1&quot;&gt;
            SELECT NVL(MAX(id),0)
            FROM databasechangelog
            WHERE author=&#039;MajorVersion&#039;
        &lt;/sqlCheck&gt;
    &lt;/preConditions&gt;
    &lt;include file=&quot;v001/2009-10-16-102.xml&quot;/&gt;
    &lt;include file=&quot;v002/2009-10-18-114.xml&quot;/&gt;
&lt;/databaseChangeLog&gt;</pre>

</div>

<h2><a>Step 14: Backport change 114 to branch 2.x</a></h2>
<div >

<p>
This is the same as step 8.
</p>

</div>

<h2><a>Step 15: Deliver release 1.1 to Solo</a></h2>
<div >

<p>
This is the same as step 9, except you will run the lb_update command instead of lb_install to update Solo to the latest version of the 1.x branch.
</p>

</div>

<h2><a>Step 16: Deliver 2.0 as a fresh install to customer Duplex</a></h2>
<div >

<p>
This is also the same as step 9, but taking the export from tag 2.0.
</p>

</div>

<h2><a>Step 17: Deliver 2.0 as an upgrade to customer Solo</a></h2>
<div >

<p>
The same export created in the previous step can be delivered to customer Solo. 
</p>

<p>
Customer lb_solo runs the <code>lb_upgrade_to_magor</code> command.
</p>

</div>

<h2><a>Summary of developer procedures</a></h2>

<h3><a>Database refactorings</a></h3>
<div >

<p>
The default action is to create a change file named &lt;date&gt;-IssueNr&gt;.xml in directory vXXX and specify the change in this file. Example filename: <code>v002/2009-12-25-305.xml</code>
</p>
<div ><table >
	<tr >
		<th > </th><th >Create</th><th >Modify</th><th >Delete</th>
	</tr>
	<tr >
		<th >Table</th><td >Default</td><td >Default</td><td >Default</td>
	</tr>
	<tr >
		<th >Sequence</th><td >Default </td><td >Default </td><td >Default </td>
	</tr>
	<tr >
		<th >View</th><td >Create new file in directory <code>latest/vw</code>. Include this file in the change file.</td><td >Modify file in directory <code>latest/vw</code>. Include this file in the change file. </td><td >See below  </td>
	</tr>
	<tr >
		<th >Trigger</th><td >Create new file in directory <code>latest/trg</code>. Include this file in the change file. </td><td >Modify file in directory <code>latest/trg</code>. Include this file in the change file. </td><td >See below</td>
	</tr>
	<tr >
		<th >Package</th><td >Create new file in directory <code>latest/pck</code>. Include this file in the change file. </td><td >Modify file in directory <code>latest/pck</code>. Include this file in the change file. </td><td >See below </td>
	</tr>
</table></div>

</div>

<h4><a>Delete object from latest directory (view, trigger, package)</a></h4>
<div >

<p>
The procedure is described here for a view, and is similar for triggers and packages.
</p>

<p>
Delete the view from the <code>latest/vw</code> directory.
</p>

<p>
Create a changeset to delete the view, with a preCondition. The preCondition prevents an error condition if the view does not exist. This will be the case if we run a changeLog on a database before the definition of the view:
</p>
<pre >    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot;&gt;
        &lt;preConditions onFail=&quot;MARK_RAN&quot;&gt;
            &lt;viewExists viewName=&quot;departments_vw&quot;&gt;
        &lt;/preConditions&gt;
        &lt;dropView viewName=&quot;departments_vw&quot;/&gt;
    &lt;/changeSet&gt;</pre>

<p>
alternative
</p>
<pre >    &lt;changeSet author=&quot;jsmith&quot; id=&quot;1&quot; failOnError=&quot;false&quot;&gt;
        &lt;dropView viewName=&quot;departments_vw&quot;/&gt;
    &lt;/changeSet&gt;</pre>

<p>
Add the changeset to the master.
</p>

<p>
Search all changelogs for inclusion of the view definition (which no longer exists). Remove this line from the relevant changelogs.
</p>

</div>

<h3><a>Other Procedures (non-refactoring)</a></h3>

<h4><a>Branch major release</a></h4>
<div >

<p>
As an example, let&#039;s assume that you are ready to branch for release 4.x.
</p>

<p>
The trunk will contain:
</p>
<ul>
<li ><div > The installation scripts for 3.x</div>
</li>
<li ><div > The upgrade changelogs to upgrade from 2.x (to 3.x)</div>
</li>
<li ><div > The upgrade changelogs to upgrade from 3.x (to 4.x)</div>
</li>
</ul>

<p>
On the trunk, perform the following actions:
</p>
<ol>
<li ><div > Create fresh install scripts for 4.x (replacing those for 3.x)</div>
</li>
<li ><div > Delete all (upgrade) changelogs from 2.x</div>
</li>
<li ><div > Create an empty changelog container for 4.x</div>
</li>
<li ><div > replace names of these containers in master.xml</div>
</li>
<li ><div > Commit</div>
</li>
</ol>

<p>
Create a branch from this revision with name "4.x". The x indicates that all further patches on 4.0 will take place on this branch. Development of 4.1 will take place on the trunk.
</p>

</div>
