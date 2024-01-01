---
id: 503
title: 'Multiple Server Names on Windows'
date: '2016-09-26T15:50:33+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=503'
permalink: /2016/09/26/multiple-server-names-on-windows/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"506";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/09/xp-system-properties-computer-name-dialog-box.jpg
categories:
    - 'Windows OS'
tags:
    - FQDN
    - NetBIOS
    - 'server name'
    - troubleshooting
---

Having multiple server names on Windows server or client is nice feature you may want to have when you are migrating applications, using servers as part of web server farms, etc.  
As opposed to the Linux hostname where names can be arbitrarily changed, names for a machine in the Windows world have multiple dependencies and if you look at the computer name dialog box you only see one single text box.  
You may think that adding aliases (CNAME) in the DNS would suffice to have your server targeted using any name you wish.  
However, since the appearance of [Strict Name Checking](https://technet.microsoft.com/en-us/library/ff660057(v=ws.10).aspx) , it is far from being enough. You must add several registry keys to your server so it knows it must accept to answer under different names.

# Adding alternate DNS name to a server

For this to happen, you must:

- add a REG\_MULTI\_SZ under: HKLM\\System\\CurrentControlSet\\Services\\DNSCache\\Parameters\\AlternateComputerNames
- one FQDN per line
- do not add the current name of the server
- run ipconfig /registerdns for the modifications to take place

The following script will do that for you in an easy way:

```
<pre class="lang:batch decode:true " title="Adding multiple FQDN names to a server">@echo off
if "%1"=="" goto :us
set p=%*
set p0=%p: =\0%
reg add HKLM\system\currentcontrolset\services\DNSCache\parameters /v AlternateComputerNames /t REG_MULTI_SZ /d %p0% /f
echo Please run ipconfig /registerdns when you want new DNS Names to appear
goto :eof
:us
echo %~n0 name1 name2... nameN
echo makes this computer responds to name1, .. nameN as DNS name in addition to its very own name.
```

# Adding alternate NetBIOS name to a server

- add a REG\_MULTI\_SZ under: HKLM\\System\\CurrentControlSet\\Services\\LanManServer\\Parameters\\OptionalNames
- one netbios name without backslashes per line
- do not add the current name of the server
- restart the server service for the modifications to take place

These steps can be done using the following script:

```
<pre class="lang:batch decode:true" title="Adding multiple NetBios names to a server">@echo off
if "%1"=="" goto :us
set p=%*
set p0=%p: =\0%
reg add HKLM\system\currentcontrolset\services\lanmanserver\parameters /v OptionalNames /t REG_MULTI_SZ /d %p0% /f
echo Please restart the Server Service when you want new NetBios Names to appear
goto :eof
:us
echo %~n0 name1 name2... nameN
echo makes this computer responds to \\name1, \\nameN as NetBIOS in addition to its very own name.
```

# Built-in command netdom

Please note that on the latest versions of Windows, you can use netdom

```
<pre class="lang:batch decode:true " title="netdom computername example">netdom computername myserver /add myothername.mydomain.local
```

The ‘myserver’ is the main short name. Every alternate name should be entered using a FQDN; THe command will create the above mentioned registry keys in both locations, add the DNS record and even the Kerberos Service Principal Name (SPN) if enough rights are given.

# Additional remark

If the changes are not working please do following :

1. Locate the following key in the registry: HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Services\\LanmanServer\\Parameters
2. add the following registry value: DisableStrictNameChecking as REG\_DWORD, its value should be 1