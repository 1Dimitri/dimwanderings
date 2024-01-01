---
id: 478
title: 'AD Forest Disaster Recovery plan'
date: '2016-08-04T12:19:31+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=478'
permalink: /2016/08/04/ad-forest-disaster-recovery-plan/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"489";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/08/repair-your-computer.png
categories:
    - 'Active Directory'
tags:
    - backup
    - 'Domain Controllers'
    - restore
    - schema
    - security
    - troubleshooting
---

Losing one domain controller isn’t a thing you appreciate, but rebuilding one DC in an running forest is easy. Losing an entire forest because of corruption or compromised assets isn’t exactly that. In this series, Let’s see how to prepare an AD Forest Disaster Recovery plan.

Hint: you’d better practice this once, because if you have to do it live for your first time, chances are you have

- inadequate backups
- missing procedure
- stressed people

## Scenario for building the AD Forest Disaster Recovery plan

Imagine you have a multi-domain forest with at least two domain controllers in each domain, which operates properly and for which you take regular backups. Someday, a system administrator discovers that the database became corrupt, or the CERT informs the company the Active Directory has been corrupt. You need to rebuild the full forest by using an old backup. How do you do it? First challenge is to know what you will need that day.

## Before it happens

Let’s be clear: you won’t be able to do anything if your Active Directory wasn’t healthy before the incident happens. It means your AD must replicate correctly and you must take regular backups.

### Backup Type

Standard backups of domain controllers are useless because we are not interested in the OS of the domain controller but in the Active Directory database contents. Therefore you must at least create system state backups. However if you only use such backups, you will only get the AD Database which means

- you have to install a new machine for its OS
- Then you restore the system state

In most cases, you won’t have time to bother with that, so you’d better create ‘bare metal backups’ which allow you to restore the full machine at once: OS including OEM drivers and the AD database

### Backup Software

Many backup software vendors will claim they are able to perform bare metal backups or system backups, but when you’ll ask them to demonstrate how they perform these steps on a domain controllers, you may have surprises. The tested way in this procedure is to use the Windows Server backup tool.

### Backup retention limit

Your backup is useless if it has been taken a long time ago. Don’t forget that the tombstone lifetime (180 days by default) will make objects disappear, and in addition that by default replication is stopped when such old data is detected.

### Procedure

The procedure has already been discussed in a previous [post](http://dimitri.janczak.net/2015/10/21/windows-server-backup-wbadmins-use-on-domain-controllers/). It will be the base for your AD Forest Disaster Recovery plan. For scheduling purposes, you may use a scheduled task which use the wbadmin command-line tool or use the scheduler in the graphical Windows Server backup tool. Both are equivalents and depend mostly on your habits in terms of scheduling and your monitoring software.

### Backup Location

Please note that you can store it locally on the machine or remotely. If stored remotely, you must be able to access the files without your AD being able to authenticate, which means another forest, or a workgroup server. If stored locally, you must access it in DS Recovery mode or be able to mount the disk by other means. (Boot with CD for physical, mount the disk file for VM)

### Hardware type considerations

Since the backup taken is a bare metal one, it include drivers and hardware dependent files. Therefore the restore must happen on the same hardware type. In most cases, the easiest way to get the same hardware is to use virtual hardware. Please however be sure that it is the same virtual hardware then:

- same Generation for Hyper-V, same VM version for VMware, etc.
- same additional components (e.g. same network card type, etc.)
- disk layout is the same (number of disks, size…)

Please bear in mind that Microsoft recommends that the PDC emulator be on a physical machine for time synchronization issues, so for AD Forests where you can afford it, prepare a mix of physical and virtual machines.

### Credentials storage

During the restore you will need an Enterprise administrator, the built-in administrator for each domain or a domain administrator per domain. Be sure to store these passwords in a safe vault and if the vault is digital, don’t make this piece of software depend on the Active Directory forest you’re trying to restore

In the next post, we’ll detail the rebuild procedure for this AD Forest Disaster Recovery plan.