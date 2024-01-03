---
id: 71
title: 'Automatically Configuring SQL Server Database mail'
date: '2014-11-17T22:27:36+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=71'
permalink: /2014/11/17/automatically-configuring-sql-server-database-mail/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"57";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
tags:
    - 'database mail'
    - job
    - operator
    - smtp
    - snippet
    - sql
    - 'SQL Server'
---

In SQL Server 2005 and higher, there is database wizard to configure the database mail, but no easy way to ask for a script of the current configuration.

Here is a little SQL script which will:

- activate Database Mail
- Create a default SMTP profile called sql\_alert\_profile and attach an SMTP account called account\_smtp\_public to it
- Send Test E-Mail
- Create one or more operators

You may wonder why there is a test mail embedded in this script whereas the contextual menu of the Database mail already provides one. It is just a way to follow the agile principles by providing automated tests.

```
<pre class="lang:tsql decode:true" data-url="Database mail configuration">-- DATABASE MAIL SETUP --

-- DJ - 20141117 - 1.1 - handles named instances 

DECLARE @servername varchar(255)
DECLARE @smtpserver varchar(255)
DECLARE @smtpport int
DECLARE @testmailrecipients varchar(255)
DECLARE @fromaddress nvarchar(128)

-- 0. Some variables  to be changed if needed
SET  @servername = CAST(SERVERPROPERTY('ServerName') AS varchar(255))
SET @smtpserver = 'mail.domain.com'
SET @smtpport  = 25
SET  @testmailrecipients = 'youraddress@domain.com'
SET @fromaddress = 'sqlserver.'+REPLACE(@servername,'\','.')+'@domain.com'
-- SMTP RFC doesn't allow backslashes in names

-- Here we go
PRINT '--- About to set up database mail --'
PRINT 'Using SMTP Server ' + @smtpserver + ':' + CAST(@smtpport AS VARCHAR(5))


-- 1. Enable the db mail feature at server level 
-- Enabling Database Mail
exec sp_configure 'show advanced options',1
reconfigure
exec sp_configure 'Database Mail XPs',1
reconfigure

-- 2.Enable service broker in the MSDB database 
-- normally you don't need it
-- USE [master]
-- GO
-- ALTER DATABASE [MSDB] SET  ENABLE_BROKER WITH NO_WAIT

--3. Creating a Profile

PRINT '-- Setting profile '
DECLARE @serverdescription nvarchar(max)
SET @serverdescription = 'Mail Service for instance ' + @servername

IF EXISTS(SELECT 1 FROM msdb.dbo.sysmail_profile WHERE  name = 'sql_alert_profile' )
exec msdb.dbo.sysmail_delete_profile_sp @profile_name= 'sql_alert_profile'

EXECUTE msdb.dbo.sysmail_add_profile_sp
@profile_name = 'sql_alert_profile',
@description =  @serverdescription

-- 4. Create a Mail account.
PRINT '-- Setting Account'

IF EXISTS(SELECT 1 FROM msdb.dbo.sysmail_account  WHERE  name = 'account_smtp_public' )
exec msdb.dbo.sysmail_delete_account_sp @account_name='account_smtp_public'

EXECUTE msdb.dbo.sysmail_add_account_sp
@account_name = 'account_smtp_public',
@email_address = @fromaddress,
@description = 'default SMTP Server',
@mailserver_name = @smtpserver,
@port=@smtpport,
@enable_ssl=0

-- 5. Adding the account to the profile
EXECUTE msdb.dbo.sysmail_add_profileaccount_sp
@profile_name = 'sql_alert_profile',
@account_name = 'account_smtp_public',
@sequence_number =1 ;

-- 6. Granting access to the profile to the DatabaseMailUserRole of MSDB
EXECUTE msdb.dbo.sysmail_add_principalprofile_sp
@profile_name = 'sql_alert_profile',
@principal_id = 0,
@is_default = 1 ;


-- 7. Sending Test Mail
PRINT ' Sending test mail to ' + @testmailrecipients
DECLARE @testsubject varchar(255)
SET @testsubject = 'A Test mail from SQL Server ' + @servername
EXECUTE msdb.dbo.sp_send_dbmail
@profile_name = 'sql_alert_profile',
@recipients = @testmailrecipients,
@body = 'Database Mail Testing... Note that it just tests the reachability of the SMTP server from the SQL Server machine, not that the SQL Agent Job can send mails',
@subject = @testsubject;
go


-- 8. allow agent to use it
exec sp_configure 'Agent XPs',1
reconfigure

USE [msdb]
GO
EXEC msdb.dbo.sp_set_sqlagent_properties @email_save_in_sent_folder=1
GO
EXEC master.dbo.xp_instance_regwrite N'HKEY_LOCAL_MACHINE', N'SOFTWARE\Microsoft\MSSQLServer\SQLServerAgent', N'UseDatabaseMail', N'REG_DWORD', 1
GO
EXEC master.dbo.xp_instance_regwrite N'HKEY_LOCAL_MACHINE', N'SOFTWARE\Microsoft\MSSQLServer\SQLServerAgent', N'DatabaseMailProfile', N'REG_SZ', N'sql_alert_profile'
GO


-- 9. reset advanced options to default
sp_configure 'show advanced options',0
reconfigure
go

-- OPERATORS --

-- Mailing lists --
USE [msdb]
GO
-- remove oldies 
-- repeat as wanted
IF EXISTS(SELECT 1 FROM msdb.dbo.sysoperators  WHERE  name = 'foobar_op' )
exec msdb.dbo.sp_delete_operator @name='foobar_op'

-- (re)add ones
 
EXEC msdb.dbo.sp_add_operator @name=N'newfoobarop', 
                @enabled=1, 
                @pager_days=0, 
                @email_address=N'newfoobarop@domain.com'

-- EOF Mail Setup --
```