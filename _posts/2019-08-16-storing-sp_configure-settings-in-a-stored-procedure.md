---
id: 956
title: 'Storing sp_configure settings in a stored procedure'
date: '2019-08-16T10:58:57+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=956'
permalink: /2019/08/16/storing-sp_configure-settings-in-a-stored-procedure/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:965;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2019/08/instmsdb-file.png
categories:
    - 'SQL Server'
    - 'SQL Server code'
tags:
    - 'instance settings'
    - sp_configure
    - xp_cmdshell
---

All SQL Server DBAS are familiar with sp\_configure to change the instance global settings. However when you need to modify temporarily those settings from within your code, such as storing sp\_configure settings in a stored procedure, the process can become awkward

You need to store the configuration setting you change but also the advanced configuration options if needed.

Fortunately, there are some little gems in Microsoft’s own code under the ‘Install’ folder of every instance. The instmsdb.sql is worth a read, as it:

- gives you some insights on how msdb is created;
- shows you that Microsoft sometimes uses good old methods like a ‘DEL’ command from the xp\_cmdshell stored procedure to simply delete a file. (In SQL Server 2016 SP2, this is line 136).

```
<pre class="lang:tsql decode:true " title="Deleting MSDB files the Microsoft way">EXECUTE(N'EXECUTE master.dbo.xp_cmdshell N''DEL ' + @device_directory + N'MSDBData.mdf'', no_output')
EXECUTE(N'EXECUTE master.dbo.xp_cmdshell N''DEL ' + @device_directory + N'MSDBLog.ldf'', no_output')
```

This DEL statement is the source of our little useful piece of code, as Microsoft wants to put the xp\_cmdshell enable status back to its previous state after issuing that DEL statement. Microsoft creates 2 temporary helpers as stored procedures called #sp\_enable\_component and #sp\_restore\_component\_state

```
<pre class="lang:tsql decode:true " title="Saving the configure state">CREATE PROCEDURE #sp_enable_component     
   @comp_name     sysname, 
   @advopt_old_value    INT OUT, 
   @comp_old_value   INT OUT 
AS
   BEGIN
   SELECT @advopt_old_value=cast(value_in_use as int) from sys.configurations where name = 'show advanced options';
   SELECT @comp_old_value=cast(value_in_use as int) from sys.configurations where name = @comp_name; 
   EXEC sp_configure 'show advanced options',1;
   RECONFIGURE WITH OVERRIDE;
   EXEC sp_configure @comp_name, 1; 
   RECONFIGURE WITH OVERRIDE;
   END
go
```

```
<pre class="lang:tsql decode:true " title="Restoring the configure state">CREATE PROCEDURE #sp_restore_component_state 
   @comp_name     sysname, 
   @advopt_old_value    INT, 
   @comp_old_value   INT 
AS
   BEGIN
   EXEC sp_configure @comp_name, @comp_old_value; 
   RECONFIGURE WITH OVERRIDE;
   EXEC sp_configure 'show advanced options',@advopt_old_value;
   RECONFIGURE WITH OVERRIDE;
   END
go
```

These are 2 good snippets which may be very handy.