---
id: 1125
title: 'Invoke-WebRequest under the System Account'
date: '2022-03-01T14:54:31+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=1125'
permalink: /2022/03/01/invoke-webrequest-under-the-system-account/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1126;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2022/03/invokewebrequest.png
categories:
    - 'Powershell Engine'
tags:
    - Invoke-WebRequest
    - 'scheduled job'
    - 'Scheduled Tasks'
    - system
---

[Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest) is a versatile Powershell CmdLet. However the way it works behind the scene creates sometimes strange behaviors.

As you may aware the HTML parser engine greatly depends on system components and [Web Browser such as Internet Explorer.](https://github.com/PowerShell/PowerShell/issues/3042)

A consequence of such a “design”, is that even on Windows Server 2016, you may have non-working Powershell scripts when running under accounts whose profile has not been initialized for the underlying system application.

As such, you will often find that a script which works flawlessly under your account doesn’t retrieve anything when launched under a scheduled task. This typically includes tasks running under the System Account.

A work-around is to run once the iexplore.exe binary under the System Account. This could be accomplished using a command such as `psexec.exe -sid "%ProgramFiles%\Internet Explorer\iexplore.exe"` under the command prompt or `./psexec.exe -sid "$Env:ProgramFiles\Internet Explorer\iexplore.exe"` under Powershell.