---
layout: default
title: Docs | DatabaseChangeLogLock Table 
---

# DATABASECHANGELOGLOCK table

Liquibase uses the DATABASECHANGELOGLOCK table ensure only one instance of Liquibase is running at one time.

Because Liquibase simply reads from the [DATABASECHANGELOG](databasechangelog_table.html) table to determine which changeSets need to run, if multiple instances of Liquibase are executed against the same database concurrently you will get conflicts.
This can happen if multiple developers use the same database instance or if there are multiple servers in a cluster which auto-run Liquibase on startup.

<table>
    <tr><th>Column</th><th>Standard&nbsp;Data&nbsp;Type</th><th>Description</th></tr>
    <tr><td>ID</td><td>INT</td><td>Id of the lock. Currently there is only one lock, but is available for future use</td></tr>
    <tr><td>LOCKED</td><td>INT</td><td>Set to "1" if the Liquibase is running against this database. Otherwise set to "0"</td></tr>
    <tr><td>LOCKGRANTED</td><td>DATETIME</td><td>Date and time that the lock was granted</td></tr>
    <tr><td>LOCKEDBY</td><td>VARCHAR(255)</td><td>Human-readable description of who the lock was granted to.</td></tr>
</table>

### Notes

If Liquibase does not exit cleanly, the lock row may be left as locked. You can clear out the current lock by running `liquibase releaseLocks` which runs `UPDATE DATABASECHANGELOGLOCK SET LOCKED=0`

