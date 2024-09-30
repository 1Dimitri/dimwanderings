---
id: 662
title: 'Remote Access to Windows'
date: '2017-04-04T14:41:16+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=662'
permalink: /2017/04/04/remote-access-windows/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"663";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/04/psexec-command-line-options.png
categories:
    - 'Windows OS'
tags:
    - DCOM
    - hacking
    - MMC
    - powershell
    - PSExec
    - 'remote access'
    - 'Scheduled Tasks'
    - troubleshooting
---

This post has for goal to list all techniques you can use to get remote access to Windows. Depending on your situation (white hacking, strange corporate DMZ, …) this could help you to test and accomplish your various goals: copying a file, running a process, etc.

# PSExec

[PsExec ](https://technet.microsoft.com/en-us/sysinternals/bb897553.aspx)is a well-known tool from Sysinternals which allows you to optionally copy a file to a server and run a command under alternate credentials including the NT AUTHORITY\\SYSTEM one. You are supposed to be local administrator of the machine and being able to access the IPC$ aka RPC endpoint mapper and associated dynamic port to do so

# WinRM / Powershell

Starting with Powershell 3.0, the use of Enter-PSSession, [Invoke-Command](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.core/invoke-command) and related [Powershell job commands](https://dimitri.janczak.net/2015/05/18/jobs-in-powershell/) allows you to run Powershell on a distant machine.

Under Windows 2008R2, the access is not enabled by default. You may use winrm quickconfig with other listed tools in this page to gain access. Starting with Windows 2012, the access is enabled by default provided your are local administrator

With WinRM comes the less-known winrs command, you can use with:

```
winrs -r:target netsh.exe winhttp show proxy
```

The advantage is that you get the command’s output back

# Startup Folder

You can copy an executable into `\\TARGET\C$\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\` and expect it to be run at logon of any process including system. You may want to try to put a `whoami` command into some batch job to see the effect.

# Run registry key

Similar to the startup folder, trick you may want to add a registry entry modeled after the following:  
`reg add \\TARGET\HKLM\Software\Microsoft\Windows\CurrentVersion\Run /v VisibleFriendlyName /t REG_SZ /d "cmd.exe"`

# AT / Scheduled Tasks

With these tools (including the [schtasks.exe](https://msdn.microsoft.com/en-us/library/windows/desktop/bb736357(v=vs.85).aspx) command-line tool), you may schedule a task to run under a set of credentials in response to different triggers: it includes date/time but also responses to events written in the log. EAsy re-occuring events can be the WInHTTP proxy, …

# MMC Application COM Object

In a blog’s article about [lateral movement techniques](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/), a security guy looked at the standard COM list of objects which comes with every Windows. Such a list can be obtained by looking at the Win32\_DCOMApplication class.

Amongst all those objects is the MMC20.Application object which interestingly has a ExecuteShellCommand taking two parameters, path of the executable and argument list. Therefore a typical piece of code would be:

```
$comobj = [Activator]::CreateInstance([Type]::GetTypeFromProgID('MMC20.Application','target'))
$comobj.Document.ActiveView.ExecuteShellCommand("C:\windows\system32\netsh.exe",$null," winhttp reset proxy","7")
```

For this to work in the Windows Advanced Security Firewall the following ports must be enabled:

- COM+ Network Access (for port 135)
- A rule to let dynamic ports for C:\\windows\\system32\\mmc.exe let in. Regular RPC-EPMAP rules from other services won’t work as they only allow traffic to svchost.exe.

If the firewall doesn’t let you in, you may receive messages such as:

Exception calling “CreateInstance” with “1” argument(s): “Retrieving the COM class factory for remote component with CLSID {49B2791A-B1AE-4C90-9B8E-E860BA07F889} from machine target failed due to the following error: 800706ba target.”

You must also be local administrator of the remote system.

The [ExecuteShellCommand method of the View object](https://msdn.microsoft.com/en-us/library/aa815396(v=vs.85).aspx) takes 4 parameters:

- the complete path to the executable
- the directory to be considered as current directory; you may want to usually pass NULL
- a list of arguments to pass along to the executable. In case there is none, you can also pass NULL
- the state of the windows (Normal, minimized, maximized). This is a set of constant. Usually you would take 7 as a value

You do not get the output back if any.