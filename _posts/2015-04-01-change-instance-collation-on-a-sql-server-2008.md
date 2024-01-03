---
id: 122
title: 'Change Instance Collation on a SQL Server 2008 or higher'
date: '2015-04-01T16:09:59+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=122'
permalink: /2015/04/01/change-instance-collation-on-a-sql-server-2008/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"57";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
tags:
    - collation
    - installation
    - setup
---

Important note before you start:

- If you perform this whereas there is already data, ALL DATA will be lost as the master, model and msdb databases are recreated during this procedure.
- In fact, all user databases will have to be re-attached and any link (like login to user) have to be re-created.
- Since system databases are re-created, jobs and every instance setting has to be re-done

For a cluster,

- stop the cluster resource holding the SQL Server Instance using the Failover cluster Manager or your favorite command-line tool<span class="anchor" id="line-6"></span>
- move the resource on the node when you intend to run the commands… or go to the node holding the resource to run the commands

Given some bugs seen on Connect, better access the SQL distribution files from a DVD or its emulation (mounted ISO, etc.)

Then issue the following command:

```
<pre class="lang:batch decode:true" title="Instance Collation Change">setup /quiet /action=REBUILDDATABASE /INSTANCENAME=NameOfInstanceWithoutNetworkResourceName /SQLSYSADMINACCOUNTS=Domain\Group /SQLCOLLATION=CollationName
```

Frequent requested CollationName are: Latin1\_General\_bin, Latin1\_General\_CI\_AS or SQL\_Latin1\_General\_CP1\_CI\_AS  
<span class="anchor" id="line-13"></span><span class="anchor" id="line-14"></span><span class="anchor" id="line-15"></span>

You can remove the /quiet to get the progress screens <span class="anchor" id="line-20"></span>and the buttons to clicks. If your instance is the default one, use MSSQLSERVER as InstanceName.

Please note that this command works for Windows Authentication mode SQL Servers. If you are in mixed mode, you may either temporarily switch to Windows Only or use the SAPWD switch to add the SA password.