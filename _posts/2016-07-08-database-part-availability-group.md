---
id: 454
title: 'Is your database part of an Availability group?'
date: '2016-07-08T20:20:32+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=454'
permalink: /2016/07/08/database-part-availability-group/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"455";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/07/SQL-Server-Availability-groups.gif
categories:
    - 'SQL Server code'
tags:
    - 'availability group'
    - DMV
    - snippet
    - 'SQL Server'
    - T-SQL
---

Today’s post is about answering a simple question I had to document a SQL Server instance. Is your database part of an Availability group? Or is it purely local to your instance?

Microsoft has of course put some Dynamic Management Views (DMV) into SQL Server to list the databases in a group. However there is no simple DMV to take the problem the reverse way. In addition, as we’ll see some Availability group related DMV list databases more than once!

We will use the following DMVs:

- The well known[ sys.databases](https://msdn.microsoft.com/en-us/library/ms178534.aspx) table
- The [sys.dm\_hadr\_name\_id\_map ](https://msdn.microsoft.com/en-us/library/hh710079.aspx)DMV which gives the name to internal ID translation for each availability group
- The[ sys.dm\_hadr\_database\_replica\_states ](https://msdn.microsoft.com/en-us/library/ff877972.aspx)which gives the status of databases in the availability groups an instance is part of

It is worthwhile to note that if a database isn’t replicated using an Availability group, it won’t appear in any \*hadr\* DMV. Therefore we will use an [OUTER JOIN](http://stackoverflow.com/questions/38549/difference-between-inner-and-outer-joins) based on the contents of sys.databases to be sure not to forget any database. A NULL value will then indicate that there is nothing found, so that this database isn’t part of any group.

Here’s our first attempt:

```
<pre class="lang:tsql decode:true" title="Tells if a database is local or part of any availability group">select  sd.name, 
(
case 
 when
  hdrs.is_primary_replica IS NULL then  'NOT REPLICATED'
 else
    'REPLICATED'
 end
) as  AGType
 from sys.databases as sd
 left outer join sys.dm_hadr_database_replica_states  as hdrs on hdrs.database_id = sd.database_id
```

Let’s add the group name:

```
<pre class="lang:tsql decode:true " title="GEt database AG group if any">select sd.name, 
(
case 
 when
  hdrs.is_primary_replica IS NULL then  'NOT REPLICATED'
 else
    'REPLICATED'
 end
) as  AGType,
COALESCE(grp.ag_name,'N/A') as AGName
 from sys.databases as sd
 left outer join sys.dm_hadr_database_replica_states  as hdrs on hdrs.database_id = sd.database_id
 left outer join sys.dm_hadr_name_id_map as grp on grp.ag_id = hdrs.group_id
```

The last part is to get to know how sys.dm\_hadr\_database\_replica\_states work. The DMV doesn’t simply return you a row per database outling the status. It does return one row when you’re on a secondary. However on a primary, it returns one row for the primary, and a row per secondary flagging with is\_local to 0 or is\_primary\_replica to 0. So we will filter by knowing if for each database there is a is\_primary\_replica to 1 row and eliminating duplicates with a DISTINCT clause.

Here’s the final code:

```
<pre class="lang:tsql decode:true" title="Tells if a database is local or part of any availability group">-- Get for a each database a row with:
-- name, AGType, AGName
-- name: name of the database
-- AGType: NOT REPLICATED/PRIMARY/SECONDARY
-- AGName: N/A if NOT REPLICATED, otherwise name of the AG Group the database is part of

select DISTINCT sd.name, 
(
case 
 when
  hdrs.is_primary_replica IS NULL then  'NOT REPLICATED'
 when exists ( select * from sys.dm_hadr_database_replica_states as irs where sd.database_id = irs.database_id and is_primary_replica = 1 ) then
	'PRIMARY'
 else
    'SECONDARY'
 end
) as  AGType,
COALESCE(grp.ag_name,'N/A') as AGName
 from sys.databases as sd
 left outer join sys.dm_hadr_database_replica_states  as hdrs on hdrs.database_id = sd.database_id
 left outer join sys.dm_hadr_name_id_map as grp on grp.ag_id = hdrs.group_id

```