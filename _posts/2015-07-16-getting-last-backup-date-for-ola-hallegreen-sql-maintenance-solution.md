---
id: 285
title: 'Getting last backup date for Ola Hallegren SQL Maintenance Solution'
date: '2015-07-16T19:59:45+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=285'
permalink: /2015/07/16/getting-last-backup-date-for-ola-hallegreen-sql-maintenance-solution/
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
    - backup
    - 'Ola Hallengren'
    - powershell
---

As already mentioned in [this post](http://dimitri.janczak.net/2015/05/27/automated-maintenance-for-sql-server-express/), [Ola ‘s solution ](https://ola.hallengren.com/)to backup your SQL Server databases is recognized as a easy-to-deploy, no stress, solution. You can easily build on top of it.

Without installing any SQL cmdlet nor looking in the table the scripts produce, here is a solution to get the latest backups for each database solely based on the files’ timestamps.

```
<pre class="lang:ps decode:true" title="Getting SQL Freshest backup time">Function Get-LatestOlaBackup {
param(
$DBRootDir
)
$RecoveryModels = Get-ChildItem $DBRootDir -Attributes Directory
$DBName = Get-Item $DBRootDir
foreach ($rm in $RecoveryModels) {
Write-Verbose "      last $rm backup for $dbRootDir"
  $NewestFile = Get-ChildItem $DBRootDir | Sort-Object -Property LastWriteTime | Select-Object -First 1
  [PSCustomObject] @{
      'TimeStamp'=$NewestFile.LastWriteTime;
      'Database'=$DBName.Name;
      'BackupType'=$rm.Name
  }
}

}

Function Get-LatestOlaBackups {
param(
[string] $RootDir
)

$databases = Get-ChildItem $RootDir -Attributes Directory
foreach ($db in $databases) {
Write-Verbose "Getting last backup for $db"
Get-LatestOlaBackup -DBRootDir $db.FullName
}

}
```

The usage is simple:

```
<pre class="lang:ps decode:true " title="OLA: Use of Get-LatestOlaBackup">Get-LatestOlaBackups -RootDir S:\SQLBackups\YourServerName | Sort TimeStamp -Desc
```

More elaborated use is to create a report to group the databases by their last backup day

```
<pre class="lang:ps decode:true " title="OLA: Group SQL backups by date">Get-LatestOlaBackups -RootDir S:\SQLBackups\YourServerName | Select @{N='DayOnly';E={$_.TimeStamp.Date}} | Group DayOnly
```

[![Get-OlaLatestBackups](http://dimitri.janczak.net/wp-content/uploads/2015/07/Get-OlaLatestBackups.png)](http://dimitri.janczak.net/wp-content/uploads/2015/07/Get-OlaLatestBackups.png)