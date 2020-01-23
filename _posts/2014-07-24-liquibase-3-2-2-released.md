---
layout: default
subnav: subnav_blog.md
title: Liquibase 3.2.2 Released
---
# Liquibase 3.2.2 Released

Liquibase 3.2.2 has been released. It is a small bugfix release to fix a checksum regression from 3.1 -> 3.2 that was not fixed in 3.2.1.

### Issues Resolved:

- <a href="https://liquibase.jira.com/browse/CORE-1938">CORE-1938</a> - defaultValueNumeric="0" or defaultValue="0" is translated to 0.0
- <a href="https://liquibase.jira.com/browse/CORE-1950">CORE-1950</a> - Checksum validation failed after Liquibase upgrade (3.1.1 -> 3.2.0)
- <a href="https://liquibase.jira.com/browse/CORE-1959">CORE-1959</a> - generateChangeLog without changeLogFile - better error message



As usual, it can be downloaded from the <a href="http://liquibase.org/download">Liquibase download page</a> and is available in the Maven repository as org.liquibase/liquibase-core.




