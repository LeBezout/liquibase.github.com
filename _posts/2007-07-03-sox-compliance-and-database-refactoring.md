---
layout: default
subnav: subnav_blog.md
title: SOX Compliance and Database Refactoring
---
Managing, tracking, and applying database changes is difficult, especially in an agile database environment where there are many changes made throughout a project's life cycle.   Even with a tool like Liquibase, It takes a lot of discipline to apply your database changes in a consistent and traceable manner.

For projects that need to deal with SOX-compliant releases the process is even more difficult because your release documentation needs to include not only how to update your database, but also how to roll back in case of a problem with the release.

It was to address this problem that we added <a href="https://www.liquibase.org/documentation/rollback.html">automatic rollback support</a> to Liquibase.  For each changeSet in your change log file, Liquibase can (usually) generate rollback SQL.  For changes that cannot be automatically undone (drop table, insert data, etc.), or if you want to override the default rollback method, you can specify a  tag containing the correct SQL.  This method of generating rollback commands works well because for most cases you don't have to do anything, and when you do have to specify rollback SQL, it is stored along side the original change.

To control the generation of generating SQL for updating a database and for rolling it back, see the <a href="https://www.liquibase.org/documentation/command_line.html">command line migrator</a> documentation.

