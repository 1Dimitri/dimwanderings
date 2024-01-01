---
id: 384
title: 'Windows Server Backup wbadmin&#8217;s use on Domain Controllers'
date: '2015-10-21T12:43:15+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=384'
permalink: /2015/10/21/windows-server-backup-wbadmins-use-on-domain-controllers/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"388";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/10/Windows_server_Backup_icon.png
categories:
    - 'Active Directory'
tags:
    - backup
    - restore
    - wbadmin
---

## Why bother with Windows Server backup in large environments?

Even in big companies, having a backup of your Active Directory forest can be challenging:

- do you know what you must backup?
- do you know if the backup infrastructure and software allows you to backup what you need
- do you know if the restore really contain what you need and not what the software publisher advertises it contain?
- did you test a full restore of your forest aka Active Directory recovery (not speaking?
- if so, did you test it under time and stress constraints?

Many Windows Administrators have discovered that their so-called ‘enterprise-grade’ solution was not able to fulfill one or more of these requirements.

## Windows Server backup built-in program to the rescue

Therefore the current Microsoft recommendation is to perform a bare metal backup with Windows Server backup. Not only you’ll get the state of your forest backed up but you’ll also have the ability to restore it from scratch only having to boot with a Windows DVD. Let’s see how to do it in a unattended fashion.

The command-line utility related to the WIndows Server Backup tool is [wbadmin](https://technet.microsoft.com/en-us/library/cc754015.aspx).

The general syntax is:

```
<pre class="lang:batch decode:true " title="Default wbadmin command">wbadmin start backup -backuptarget:D:
```

You may wonder what is the difference between a wbadmin start backup and a wbadmin start systemstatebackup command.

The second one is only used for a systemstatebackup which contains the backups related to the data which does not belong to a specific drive, e.g. on a Domain Controller you’ll find the Active Directory Database and Sysvol share contents. However for some mysterious reasons known only to the developers of wbadmin, you are not able to create a bare metal backup set.

In order to do a bare metal backup you have to use the following command as abase

```
<pre class="lang:batch decode:true " title="Standard wbadmin bare metal backup basics">wbadmin start backup -allcritical - backuptarget:Q:
```

However you can make this command better by adding two flags:

```
<pre class="lang:default decode:true " title="Better wbadmin bare metal restore command">wbadmin start backup -allcritical -systemstate -backuptarget:Q: -quiet
```

You may easily deduct that -quiet will avoid messages which are pretty useless for our automated tasks.

SystemState could be seen as not needed because it is included in the “AllCritical” switch.

However adding it allows you to select **at restore time** a simple ‘systemstate’ restore from within the machine or use your backup a as bare metal restore DVD.

Should you have no other backup taken by another product (IBM TSM, Symantec NetBackup, …), you can also add the -vssfull flag. If this flag is set, the log are cleared. If it isn’t set, the system does not record that a backup has been taken, so the other products aren’t confused. This is the equivalent of not using COPY\_ONLY is SQL Server backups for example.

To finish this setup, you may want to use the following task to recreate a recurring task. You just need to change the drive letter (E:) and the time the backup occur between your different Domain Controllers.

```
<pre class="lang:xhtml decode:true " title="Scheduled task for Domain Controller backup"><?xml version="1.0" encoding="UTF-16"?>
<Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
  <RegistrationInfo>
    <Date>2015-10-21T14:20:42.0521288</Date>
    <Author>Dimitri</Author>
  </RegistrationInfo>
  <Triggers>
    <CalendarTrigger>
      <StartBoundary>2015-10-24T04:18:57</StartBoundary>
      <Enabled>true</Enabled>
      <ScheduleByDay>
        <DaysInterval>1</DaysInterval>
      </ScheduleByDay>
    </CalendarTrigger>
  </Triggers>
  <Principals>
    <Principal id="Author">
      <UserId>S-1-5-18</UserId>
      <RunLevel>HighestAvailable</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>P3D</ExecutionTimeLimit>
    <Priority>7</Priority>
  </Settings>
  <Actions Context="Author">
    <Exec>
      <Command>C:\Windows\System32\wbadmin.exe</Command>
      <Arguments>start backup -allcritical -systemstate -backuptarget:E: -quiet</Arguments>
    </Exec>
  </Actions>
</Task>
```