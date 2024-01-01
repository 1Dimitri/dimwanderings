---
id: 221
title: 'Recycling more&#8230; SQL Server (Agent) Logs'
date: '2015-06-23T15:21:47+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=221'
permalink: /2015/06/23/recycling-more-sql-server-agent-logs/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:57;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
tags:
    - 'Error log'
    - Logs
    - snippet
    - 'SQL Agent'
    - 'SQL Agent Job'
---

Some of the recommendations for better administering the logs of a SQL Server include

- adding more logs than the default value of 6
- executing a periodic rotation at a given point in a time instead of relying of the system only

Here are then a few snippets to perform the two tasks.

First, the number of logs is set up through a registry entry. This is the official script that SQL Server management studio writes when you push the ‘script’ button on the properties window of the SQL Server Logs entry in the Management folder.

```
<pre class="lang:tsql decode:true " title="Changing the number of SQL Server logs">USE [master]
GO
EXEC xp_instance_regwrite N'HKEY_LOCAL_MACHINE', N'Software\Microsoft\MSSQLServer\MSSQLServer', N'NumErrorLogs', REG_DWORD, 30
GO
```

I choose a value of 30 that gives me roughly an entry per day during one month. Most recommendations will make this value vary between 7 (one week with no restart) and 99 (compliance paranoid freaks).

And now here is a SQL Server agent job that will do a SQL Server log and SQL Server agent log recycle at midnight:

The first if statement allows you to put that job in a non-standard predefined category

```
<pre class="lang:tsql decode:true" title="SQL Agent job to recycle SQL logs at midnight ">BEGIN TRANSACTION
DECLARE @ReturnCode INT
SELECT @ReturnCode = 0

IF NOT EXISTS (SELECT name FROM msdb.dbo.syscategories WHERE name=N'Misc Maintenance' AND category_class=1)
BEGIN
EXEC @ReturnCode = msdb.dbo.sp_add_category @class=N'JOB', @type=N'LOCAL', @name=N'Misc Maintenance'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

END

DECLARE @jobId BINARY(16)
EXEC @ReturnCode =  msdb.dbo.sp_add_job @job_name=N'Maint - Cycle Logs', 
                @enabled=1, 
                @notify_level_eventlog=0, 
                @notify_level_email=0, 
                @notify_level_netsend=0, 
                @notify_level_page=0, 
                @delete_level=0, 
                @description=N'No description available.', 
                @category_name=N'Misc Maintenance', 
                @owner_login_name=N'sa', @job_id = @jobId OUTPUT
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'Cycle Agent Log', 
                @step_id=1, 
                @cmdexec_success_code=0, 
                @on_success_action=3, 
                @on_success_step_id=0, 
                @on_fail_action=3, 
                @on_fail_step_id=0, 
                @retry_attempts=0, 
                @retry_interval=0, 
                @os_run_priority=0, @subsystem=N'TSQL', 
                @command=N'EXEC  sp_Cycle_Agent_ErrorLog



 ', 
                @database_name=N'msdb', 
                @flags=0
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback

EXEC @ReturnCode = msdb.dbo.sp_add_jobstep @job_id=@jobId, @step_name=N'Cycle SQL Server Log', 
                @step_id=2, 
                @cmdexec_success_code=0, 
                @on_success_action=1, 
                @on_success_step_id=0, 
                @on_fail_action=2, 
                @on_fail_step_id=0, 
                @retry_attempts=0, 
                @retry_interval=0, 
                @os_run_priority=0, @subsystem=N'TSQL', 
                @command=N'EXEC sp_Cycle_ErrorLog
', 
                @database_name=N'master', 
                @flags=0
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_update_job @job_id = @jobId, @start_step_id = 1
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobschedule @job_id=@jobId, @name=N'Maint - Midnight roll', 
                @enabled=1, 
                @freq_type=4, 
                @freq_interval=1, 
                @freq_subday_type=1, 
                @freq_subday_interval=0, 
                @freq_relative_interval=0, 
                @freq_recurrence_factor=0, 
                @active_start_date=20150622, 
                @active_end_date=99991231, 
                @active_start_time=0, 
                @active_end_time=235959 
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
EXEC @ReturnCode = msdb.dbo.sp_add_jobserver @job_id = @jobId, @server_name = N'(local)'
IF (@@ERROR <> 0 OR @ReturnCode <> 0) GOTO QuitWithRollback
COMMIT TRANSACTION
GOTO EndSave
QuitWithRollback:
    IF (@@TRANCOUNT > 0) ROLLBACK TRANSACTION
EndSave:

GO
```

Do not forget that logs will be also rotated if you restart the corresponding.-service