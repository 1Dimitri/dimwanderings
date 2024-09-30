---
id: 163
title: 'Moving Active Directory database related files to another location'
date: '2015-05-21T19:06:09+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=163'
permalink: /2015/05/21/moving-active-directory-database-related-files-to-another-location/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"170";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/05/move-folder.jpg
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - 'file move'
    - maintenance
    - ntds
---

Here’s your problem for the day: your active directory files are on the system drive, perhaps because you inherited some Active Directory domain controllers, perhaps because a junior Windows system administrator did the DCPromo… You’re a professional Windows system administrator and you want to have them located elsewhere, say D:

The process is documented at Technet’s but requires two separate duties:

- Moving the databases files per se, namely the ntds.dit and jet files; Starting with WIndows 2008, this is quick and easy, and downtimes may be unnoticed if you are quick enough
- Moving the SYSVOL replicated share, takes more time and far from being simple

<span style="text-decoration: underline;">*Moving the database and or logs files*</span>

It is documented under [Move the directory Database and Log Files to a Local Drive](https://technet.microsoft.com/en-us/library/cc816720%28v=ws.10%29.aspx) and involve the following steps

1. Disable your antivirus (as it may not guess what kind of files we are moving); that’s not in Microsoft’s procedure BTW
2. create two elevated command prompt if your logs and database are separated; and for each of them go the actual location of the database or the log, e.g.
3. cd C:\\ADLogs or cd C:\\ADDatabase
4. Prepare a notepad with the following contents so you ‘ve got the two ntdsutil scripts: ```
    <pre class="lang:batch decode:true" title="ntdsutil script to move database files">activate instance ntds
    files
    move db to D:\ntds
    quit
    quit
    ```
    
    ```
    <pre class="lang:batch decode:true" title="ntdsutil script to move log files">activate instance ntds
    files
    move logs to D:\ntds
    quit
    quit
    ```
    
    where d:\\ntds is the target folder where you want the files to be moved to. I do not follow the practice to separate logs from databases because in most cases the speed of the storage is no longer the culprit and nowadays you likely have no influence over it (Virtual machines datastores, etc.),. What I want to achieve is separating the Active Directory files from the OS files.
5. Stop the NTDS Service
6. run the first script from the database folder and look for error messages
7. run the second script from the logs folder and still look for error messages
8. Start the NTDS Service
9. Look into the Event Viewer for issues

The Microsoft documentation tells you to perform integrity and security checks but the output of the ntdsutil commands are rather verbose and safe. In addition, you may obtain the JET\_errOutOfSessions error message when doing so, but the hotfix outlined by Microsoft is no longer applicable to latest releases of Windows 2008R2.

*<span style="text-decoration: underline;">Moving the Sysvol share</span>*

The sysvol share move takes longer and exists in two flavors not related to the OS you’re running although what Microsoft says, but to the way the replication is done: are you FRS or DFS-R?

- If you are doing NTFRS, use the [Windows 2003 version](https://technet.microsoft.com/en-us/library/cc786035%28v=ws.10%29.aspx) even you are on Windows 2008 or higher
- If you are doing DFS-R, follow the [Windows 2008 version](https://technet.microsoft.com/en-us/library/cc816594%28v=ws.10%29.aspx)

The two procedures are OK, but you have a few caveats in the Windows 2003 Version on Windows 2008.

- Be sure to fill in the table located there

| Parameter | Current Value | New Value |
|---|---|---|
| fRSRootPath |  |  |
| fRSStagingPath |  |  |
| Sysvol parameter in registry |  |  |
| Sysvol junction |  |  |
| Staging junction |  |  |

- In order to fill in this table, you can use Powershell by doing:

```
<pre class="lang:ps decode:true" title="Getting SysVol registry key">cd HKLM:\System\CurrentControlSet\Services\NetLogon\Parameters
gi .
```

```
Import-Module ActiveDirectory
cd AD:
cd '.\DC=domain,DC=fqdn' # CHange with your Domain
cd '.\CN=Name of your DC'
cd '.\CN=NTFRS Subscriptions'
cd '.\CN=Domain System Volume'
gi . -Prop *
```

- Assuming you are moving from C:\\Windows\\sysvol to D:\\Sysvol, you’ll obtain something like this

| Parameter | Current Value | New Value |
|---|---|---|
| fRSRootPath | C:\\Windows\\SYSVOL\\domain | D:\\SYSVOL\\domain |
| fRSStagingPath | C:\\Windows\\SYSVOL\\staging\\domain | D:\\SYSVOL\\staging\\domain |
| Sysvol parameter in registry | C:\\Windows\\SYSVOL\\sysvol | D:\\SYSVOL\\sysvol |
| Sysvol junction | C:\\Windows\\SYSVOL\\domain | D:\\SYSVOL\\domain |
| Staging junction | C:\\Windows\\SYSVOL\\staging\\domain | D:\\SYSVOL\\staging\\domain |

1. Stop the NTFRS Service
2. Copy the C:\\Windows\\Sysvol folder using File Explorer
3. Change the registry key HKLM\\System\\CurrentControlSet\\Services\\NetLogon\\Parameters\\Sysvol to the new location
4. Change both fRSRootPath and fRSStagingPath in the NTFRS object
5. Change both junctions (On the C: and the D: drive)
6. instead of using linkd, you may want to use the junction utility from the SysInternals tools. In this case you must remove the junction and then create it again ```
    <pre class="lang:batch decode:true" title="junction commands for sysvol">junction -d domain.fqdn
    junction domain.fqdn D:\Sysvol\domain
    ```
    
    ```
    <pre class="lang:batch decode:true" title="junction commands for staging">junction -d domain.fqdn
    junction domain.fqdn D:\Sysvol\staging\domain
    ```
7. Change the BurFlags to a non-authorative restore
8. Restart the ntfrs service
9. Perform the various dcdiag tests. If you missed one junction on scripts, NETLOGON won’t show up, just recreate it and restart the netlogon service.
