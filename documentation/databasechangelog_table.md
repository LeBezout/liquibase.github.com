---
layout: default
title: DatabaseChangeLog Table | Liquibase Docs
---

# DATABASECHANGELOG table

Liquibase uses the `DATABASECHANGELOG` table to track which changeSets have been run.

The table tracks each changeSet as a row, identified by a combination of the "id", "author", and "filename" columns.

There is no primary key on the table. This is to avoid any database-specific restrictions on key lengths. The composite of "id", "author", and "filename" 
is unique across all rows of the table.

<table>
    <tr><th>Column</th><th>Standard&nbsp;Data&nbsp;Type</th><th>Description</th></tr>
    <tr><td>ID</td><td>VARCHAR(255)</td><td>Value from the changeSet "id" attribute</td></tr>
    <tr><td>AUTHOR</td><td>VARCHAR(255)</td><td>Value from the changeSet "author" attribute</td></tr>
    <tr><td>FILENAME</td><td>VARCHAR(255)</td><td>Path to the changelog. This may be an absolute path or a relative path depending on how the changelog 
            was passed to Liquibase. For best results, it should be a relative path. The `logicalFilePath` attribute can be used
            [on the changelog](/documentation/databasechangelog.html) or [on individual changeSets](/documentation/changeset.html).</td></tr>
    <tr><td>DATEEXECUTED</td><td>DATETIME</td><td>Date/time of when the changeSet was executed. Used with ORDEREXECUTED to determine rollback order</td></tr>
    <tr><td>ORDEREXECUTED</td><td>INT</td><td>Order that the changeSets were executed. Used in addition to DATEEXECUTED to ensure order is correct even when the databases datetime supports poor resolution.<br><br>NOTE: The values are only guaranteed to be increasing within an individual update run. There are times where they will restart at zero.</td></tr>
    <tr><td>EXECTYPE</td><td>VARCHAR(10)</td><td>Description of how the changeSet was executed. Possible values include "EXECUTED", "FAILED", "SKIPPED", "RERAN", and "MARK_RAN"</td></tr>
    <tr><td>MD5SUM</td><td>VARCHAR(35)</td><td>Checksum of the changeSet when it was executed. Used on each run to ensure there have been no unexpected changes to changeSet in the changelog file.</td></tr>
    <tr><td>DESCRIPTION</td><td>VARCHAR(255)</td><td>Short auto-generated human readable description of changeSet</td></tr>
    <tr><td>COMMENTS</td><td>VARCHAR(255)</td><td>Value from the changeSet "comment" attribute</td></tr>
    <tr><td>TAG</td><td>VARCHAR(255)</td><td>Tracks which changeSets correspond to tag operations.</td></tr>
    <tr><td>LIQUIBASE</td><td>VARCHAR(20)</td><td>Version of Liquibase used to execute the changeSet</td></tr>
    <tr><td>CONTEXTS</td><td>VARCHAR(255)</td><td>Context(s) used to execute the changeSet</td></tr>
    <tr><td>LABELS</td><td>VARCHAR(255)</td><td>Label(s) used to execute the changeSet</td></tr>
    <tr><td>DEPLOYMENT_ID</td><td>VARCHAR(10)</td><td>Changesets deployed together will have the same unique identifier</td></tr>
</table>


