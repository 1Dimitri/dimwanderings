---
id: 118
title: 'Enabling ETW tracing for NTLM issues'
date: '2015-03-10T08:17:38+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=118'
permalink: /2015/03/10/enabling-etw-tracing-for-ntlm-issues/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"148";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/03/etw-architecture.png
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - ETW
    - 'event tracing'
    - NTLM
    - troubleshooting
---

ETW (Event Tracing for WIndows) is said to be powerful by Microsoft, but the setup of the various providers can be tedious because the documentation often lacks examples for the specific provider you desesperately need.

Here is a script to start recording NTLM authentication traces on a Domain Controller, in the existing directory of your choice

```
<pre class="lang:batch decode:true  " title="Enabling ETW tracing on Domain Controller for NTLM Authentication">@echo off
if "%1"=="" goto :usage
if not exist %1 goto :usage
ECHO These commands will enable tracing:
@echo on
logman create trace "ds_ds" -ow -o %1\%computername%_ds.etl -p {1C83B2FC-C04F-11D1-8AFC-00C04FC21914} 0xffffffffffffffff 0xff -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 4096 -ets
logman update trace "ds_ds" -p {8E598056-8993-11D2-819E-0000F875A064} 0xffffffffffffffff 0xff -ets
logman update trace "ds_ds" -p {F33959B4-DBEC-11D2-895B-00C04F79AB69} 0xffffffffffffffff 0xff -ets
logman update trace "ds_ds" -p {24DB8964-E6BC-11D1-916A-0000F8045B04} 0xffffffffffffffff 0xff -ets
@echo off
echo
ECHO Reproduce your issue and enter any key to stop tracing
@echo on
pause
logman stop "ds_ds" -ets
@echo off
echo Tracing has been captured and saved successfully at c:\ds_ds.etl
pause
goto :eof
:usage
echo %~nx0 directory
echo records the NTLM ETW trace in the existing directory as %computername%_ds.etl
```