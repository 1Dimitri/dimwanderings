---
id: 202
title: 'Automated maintenance for SQL Server Express'
date: '2015-05-27T19:27:01+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=202'
permalink: /2015/05/27/automated-maintenance-for-sql-server-express/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";b:0;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - 'SQL Server'
tags:
    - maintenance
    - 'Ola Hallengren'
    - powershell
    - 'scheduled job'
    - 'SQL Server Express'
---

One of the drawbacks of using Express Editions of SQL Server is the lack of working SQL Agent Jobs. In particular when you are using [Ola Hallengren’s excellent maintenance solution](https://ola.hallengren.com/) for backups and other maintenance tasks. [His FAQ](https://ola.hallengren.com/frequently-asked-questions.html) just suggests that you use sqlcmd to execute the stored procedures, but doesn’t give much details. It can be handy to use powershell scripts to embed that call to sqlcmd and perform additional work based on the result code.

```
<pre class="lang:ps decode:true " title="Powershell to launch backup using Ola's Hallegren solution">$SmtpServer = 'mysmtp@domain.fqdn' #or use $PSEmailServer directly
$Sender = "sqlserver@domain.fqdn"
$Recipients = @("operators@domain.fqdn")
# Path for SQL Server 2008 (R2)
$sqlcmd = Join-Path $Env:ProgramFiles 'Microsoft SQL Server\100\Tools\Binn\SQLCMD.EXE'
$SQLInstance = 'SERVER\INSTANCE'
$SpSql=$SqlInstance.split('\')
$ServerNAme=$SpSql[0]
$AppName='Some Friendly Name'
$TargetBackupFolder = "'D:\SQLBkp'"
$Batch = "EXECUTE [dbo].[DatabaseBackup] @Databases = 'ALL_DATABASES', @Directory = N$($TargetBackupFolder), @BackupType = 'FULL', @Verify = 'Y', @CleanupTime = 48, @CheckSum = 'Y', @LogToTable = 'Y'"
& $sqlcmd -E -S $SQLInstance -d cs_helper -b -Q $Batch

$SQLQueryResult = $LASTEXITCODE
if ($SQLQueryResult -eq 0)
{
        Send-MailMessage -SmtpServer $SmtpServer -From $Sender -To $Recipients -Subject "[$ServerName][$AppName][OK] Bkp - ALL DATABASES - FULL - DAILY" -Body "All Databases backups from $SqlInstance have been done successfully. What a good day!"
        Return $SQLQueryResult
}
else
{
        Send-MailMessage -SmtpServer $SmtpServer -From $Sender -To $Recipients -Subject "[$ServerName][$AppName][KO] Bkp - ALL DATABASES - FULL - DAILY" -Body "All Databases backups from $SqlInstance failed. Please check the CommandLog table to know more about the error."
        Return $SQLQueryResult
}
```

Should you need to perform other maintenance tasks, you can replace the execute line with one of the following statements:

```
<pre class="lang:ps decode:true " title="Vatrious statements for Ola Hallegreen's solution"># backup
$Batch = "EXECUTE [dbo].[DatabaseBackup] @Databases = 'ALL_DATABASES', @Directory = N$($TargetBackupFolder), @BackupType = 'FULL', @Verify = 'Y', @CleanupTime = 48, @CheckSum = 'Y', @LogToTable = 'Y'"

# database check
$Batch = "EXECUTE [dbo].[DatabaseIntegrityCheck] @Databases = 'ALL_DATABASES', @LogToTable = 'Y'"

# Index optimization
$Batch = "EXECUTE [dbo].[IndexOptimize] @Databases = 'ALL_DATABASES', @LogToTable = 'Y'"

# Statistics update
$Batch = "EXECUTE [dbo].[IndexOptimize] @Databases = 'ALL_DATABASES',@FragmentationLow = NULL,@FragmentationMedium = NULL,@FragmentationHigh = NULL,@UpdateStatistics = 'ALL', @LogToTable = 'Y'"
```

You can then schedule those powershell scripts using either the scheduled tasks or the PSScheduledJobs.