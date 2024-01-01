---
id: 413
title: 'Event Log Rights for Non-Administrators'
date: '2016-01-21T15:12:41+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=413'
permalink: /2016/01/21/event-log-rights-for-non-administrators/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"416";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/01/Setup-Event-Log-Rights-and-settings.png
categories:
    - 'Windows OS'
tags:
    - 'event log'
    - 'event tracing'
    - SDDL
---

Granting event log rights to non-administrators wsa challenging in Windows 2003, becomes easier in Windows 2008 (R2). However fine-tuned access still requires playing with Security Descriptors [reading and writing](https://msdn.microsoft.com/en-us/library/windows/desktop/aa379567%28v=vs.85%29.aspx).

## Event Log Rights Case #1: Read Access only

For Windows 2008, If you just want to grant regular read access, the built-in “Event Log Readers” group is fine. Just put your user(s) into that group.

## Event Log Rights Case #2: Read-Write (or other) Access

If you need to grant read/write access or grant access to other groups/users than the “Event Log Readers” you must create your own SDDL descriptor for each log you want to give access to.

Let’s take the example of the application log. To get the current list of authorized access you can type in the following command:

```
<pre class="lang:batch decode:true " title="Getting current Security Descriptor for Application Log as text">wevtutil gl application > application-log-settings.txt
```

Alternatively you can get a XML output with:

```
<pre class="lang:batch decode:true " title="Getting current Security Descriptor for Application Log as XML">wevtutil gl application /f:XML > application-log-settings.xml
```

The line which is of interest is channelAccess. By default, you get the following entry:

```
<pre class="lang:batch decode:true" title="wevtutil gl output">name: application
enabled: true
type: Admin
owningPublisher:
isolation: Application
channelAccess: channelAccess: O:BAG:SYD:(A;;0xf0007;;;SY)(A;;0x7;;;BA)(A;;0x3;;;BO)(A;;0x5;;;SO)(A;;0x1;;;IU)(A;;0x3;;;SU)(A;;0x1;;;S-1-5-3)
(A;;0x2;;;S-1-5-33)(A;;0x1;;;S-1-5-32-573)
  logFileName: %SystemRoot%\System32\Winevt\Logs\application.evtx
  retention: false
  autoBackup: false
  maxSize: <Size Set in your GPO>
publishing:
  fileMax: 1
```

The S-1-5-32-573 represents the “Event Log Readers Group”, as mentioned in the [well-known groups &amp; users list](https://support.microsoft.com/en-us/kb/243330), and 0x1 means it has read access only.

If you want to add a read/write access for one user or group, just get its SID and grant him the 0x3 right. To get the SID you can use pssid from Sysinternals or Get-ADUser / Get-ADGroup cmdlets in powershell:

```
<pre class="lang:ps decode:true" title="Get-ADGroup output example">PS > Get-ADGroup My-Log-Admins

DistinguishedName : CN=My-Log-Admins,OU=Admin Accounts,DC=domain,DC=dom
GroupCategory     : Security
GroupScope        : Global
Name              : My-Log-Admins
ObjectClass       : group
ObjectGUID        : deadbeef-dead-beef-dead-beefdeadbeef
SamAccountName    : My-Log-Admins
SID               : S-1-5-21-xxxxxxxxxx-yyyyyyyyyy-zzzzzzzzzz-rrrr
```

Then use the wevtutil sl command and its /ca switch to override the channelAccess value:

```
<pre class="lang:batch decode:true" title="wevtutil set security descriptor example">C:\Windows\system32>wevtutil sl application /ca:O:BAG:SYD:(A;;0xf0007;;;SY)(A;;0x7;;;BA)(A;;0x3;;;BO)(A;;0x5;;;SO)(A;;0x1;;;IU)(A;;0x3;;;SU)(A;;0x1;;;S-1-5-3)(A;;0x2
;;;S-1-5-33)(A;;0x1;;;S-1-5-32-573)(A;;0x3;;;S-1-5-21-xxxxxxxxxx-yyyyyyyyyy-zzzzzzzzzz-rrrr)
```

Put everything on a single line! You can chheck the change has been made by re-issuing the gl switch.

## Event Log Rights Case #3: Security Log case

If you ‘just’ need read and write rights on the security log, you could also assign the privilege ‘Managing and Auditing the Security log’. However this gives additional rights to the user, like setting the audit descriptors (Success, FAilure) on objects. That’s often a bit too much.