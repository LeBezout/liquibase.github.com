---
layout: default
title: Trimming ChangeLog | Liquibase Docs
---

# Trimming ChangeLog Files

For people who have used Liquibase for a long time, a common question they have is how to clear out a changelog file that has gotten unwieldy.

The standard process for using Liquibase is to append individual change sets to your changelog file for each database change you need to make. Over time those changes can build up to thousands of entries, many of which are now redundant (create a table and later drop it) or inefficient (create a table, then add columns individually vs. just creating the table with all the columns). What is the best way to simplify all that cruft that has built up?

My first response is always "Do you really need to simplify it?" You built up that changelog over a long period of time and you have ran it and tested it countless times. Once you start messing with the changelog file you are introducing risk which has a cost of its own. Does whatever performance or file size concerns you have really outweigh the risk of messing with a script that you know works?

If it is worth the risk, why is it work the risk? Sometimes the problem is that your changelog file has just gotten so large that your editor chokes on it, or you get too many merge conflicts. The best way to handle this is to simply break up your changelog file into multiple files. Instead of having a single changelog.xml file with everything in it, create a master.changelog.xml file which uses the tag to reference other changelog files.

{% highlight xml %}
<databaseChangeLog
            xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">
    <include file="com/example/news/news.changelog.xml"/>
    <include file="com/example/directory/directory.changelog.xml"/>
</databaseChangeLog>
{% endhighlight %}

When you run `liquibase update` against the master.changelog.xml file, changeSets in com/example/news/news.changelog.xml will run and then the changeSets in com/example/directory/directory.changelog.xml will run. You can break up changeSets in whatever manner works best for you. Some break them up by feature, some break them up by release. Find what works best for you.

Other times, the problem is that `liquibase update` is taking too long. Liquibase tries to be as efficient as possible when comparing the contents of the DATBASECHANGELOG table with the current changelog file and even if there are thousands of already ran changeSets, an "update" command should take just seconds to run. If you are finding that update is taking longer than it should, watch the Liquibase log to determine why. Perhaps there is an old runAlways="true" changeSet that no longer needs to run or there are preconditions which are no longer needed. Running Liquibase with --logLevel=INFO or even --logLevel=DEBUG can give additional output which can help you determine which changeSets are slow. Once you know what is slowing down your update, try to alter just those changeSets rather than throwing out the whole changelog and starting from scratch. You will still want to retest your changelog in-depth, but it is a far less risky change.

For other people, they find that `liquibase update` works well for incremental updates, but creating a database from scratch takes far too long. Again I would ask "is that really a problem?" Are you re-creating databases often enough that the risk of a change to the creation script makes sense? If you are, your first step should be to look for problem changeSets as described above. Databases are fast, especially when they are empty. Even if you create a table only to drop it again that is usually just a few milliseconds of overhead and not worth optimizing. The biggest performance bottlenecks in creating a database are usually indexes, so start with them. If you are creating and updating indexes frequently in your creation process, you may be able to combine those changeSets into something more efficient.

When you need to surgically alter your existing changeSets, remember how Liquibase works: each changeSet has an "id", an "author", and a file path which together uniquely identifies it. If the DATABASECHANGELOG table has an entry for that changeSet it will not run it. If it has an entry, it throws an error if the checksum for the changeSet in the file doesn't match what was stored on the last run.

How you modify your existing changeSets will also depend on your environment and where in the changelog the problem changeSets are. If you are modifying changeSets that have been applied to all of your environments and are now only used on fresh database builds you can treat them differently than if they have been applied to some databases but not yet to others.

To merge or modify existing changeSets you will be doing a combination of editing existing changeSets, removing old changeSets, and creating new ones.

Removing unneeded changeSets is easy because Liquibase doesn't care about DATABASECHANGELOG rows with no corresponding changeSets. Just delete out of date changeSets and you are done. For example, if you have a changeSet that creates the table "cart" and then another that drops it, just remove both changeSets from the file. You must make sure, however, that there are no changeSets between the create and the delete that make use of that table or they will fail on a fresh database build. That is an example of how you are introducing risk when changing your changelog file.

Suppose instead you have a "cart" table that is created in one changeSet, then a "promo_code" column is created in another and an "abandoned" flag is created in another.

{% highlight xml %}
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                       xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">
    <changeSet author="nvoxland" id="1">
        <createTable tableName="cart">
            <column name="id" type="int"/>
        </createTable>
    </changeSet>

    <changeSet author="nvoxland" id="2">
        <addColumn tableName="cart">
            <column name="promo_code" type="varchar(10)"/>
        </addColumn>
    </changeSet>

    <changeSet author="nvoxland" id="3">
        <addColumn tableName="cart">
            <column name="abandoned" type="boolean"/>
        </addColumn>
    </changeSet>

</databaseChangeLog>
{% endhighlight %}

One option would be to combine everything into a new changeSet using the existing id="1" and delete the other changeSets.

{% highlight xml %}
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">
    <changeSet author="nvoxland" id="1">
        <validCheckSum>7:f24b25ba0fea451728ffbade634f791d</validCheckSum>
        <createTable tableName="cart">
            <column name="id" type="int"/>
            <column name="promo_code" type="varchar(10)"/>
            <column name="abandoned" type="boolean"/>
        </createTable>
    </changeSet>
</databaseChangeLog>
{% endhighlight%}

This will work well if all existing databases have the cart table with the promo_code and abandoned columns already added. Running Liquibase against existing databases just sees that id="1" already ran and doesn't do anything new. Running Liquibase against a blank database will create the cart table with all the columns right away. Notice that we had to add the flag or existing databases will thow an error saying that id="1" has changed since it was run. Just use the checksum in the error message in the validCheckSum tag to mark that you know it changed and the new value is OK.

If you have some databases where the promo_code and/or abandoned columns have not yet been added, update the original createTable as before, but use preconditions with onFail="MARK_RAN" to handle cases where the old changeSet ran while still not adding the columns again if the new changeSet ran.

{% highlight xml %}
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.8.xsd">
    <changeSet author="nvoxland" id="1">
        <validCheckSum>7:f24b25ba0fea451728ffbade634f791d</validCheckSum>
        <createTable tableName="cart">
            <column name="id" type="int"/>
            <column name="promo_code" type="varchar(10)"/>
            <column name="abandoned" type="boolean"/>
        </createTable>
    </changeSet>

    <changeSet author="nvoxland" id="2">
        <preConditions onFail="MARK_RAN">
            <not><columnExists tableName="cart" columnName="promo_code"/></not>
        </preConditions>
        <addColumn tableName="cart">
            <column name="promo_code" type="varchar(10)"/>
        </addColumn>
    </changeSet>

    <changeSet author="nvoxland" id="3">
        <preConditions onFail="MARK_RAN">
            <not><columnExists tableName="cart" columnName="abandoned"/></not>
        </preConditions>
        <addColumn tableName="cart">
            <column name="abandoned" type="boolean"/>
        </addColumn>
    </changeSet>

</databaseChangeLog>
{% endhighlight %}

Now, on existing databases that have all 3 changeSets already ran, Liquibase will just continue on as before. For existing databases that have the old cart definition, it will see that the columns don't exist for id="2" and id="3" and execute then as usual. For blank databases, it will create the table with the promo_code and abandoned columns and then in id="2" and id="3" it will see that they are already there and mark that they have ran without re-adding the columns. A word of warning, however: using preconditions will add a performance overhead to your update executions and are ignored in updateSQL mode because Liquibase cannot know how applicable they are when changeSets have not actually executed. For that reason it is best to avoid them if possible, but definitely use them when needed. Preconditions also add complexity to your changelog which will require additional testing so keep that in mind when deciding whether to modify your changelog logic. Sometimes it is easiest and safest to wait until all your databases have the columns and then modify the changeSets to avoid the preconditions.

The cart/promo_code/abandoned example shows some basic patterns you can use when modifying existing changeSets. Similar patters can be used to optimize whatever your bottlenecks are. Just remember when you change one changeSet, it can affect other changeSets below which may need to be modified as well. This can easily spider out of control so be mindful of what you are doing.

If you end up finding that it will work best to completely restart your changelog, see "[Introducing Liquibase into an existing project](existing_project.html)" which describes how to add Liquibase to an existing project (even if that project was previously managed by Liquibase).
