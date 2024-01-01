---
id: 1117
title: 'Slipstreaming SQL Server 2019'
date: '2022-02-07T10:26:09+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=1117'
permalink: /2022/02/07/slipstreaming-sql-server-2019/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1118;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2022/02/SQL-Server-2019-illustration.png
categories:
    - batch
    - 'SQL Server'
tags:
    - automation
    - slipstream
    - unattended
---

A while ago, I published a post to [slipstream the SQL Server 2008 media](https://dimitri.janczak.net/2014/10/12/slipstreaming-sql-server-2008-2008-r2-service-packs/). It is time to update it for SQL Server 2019.

Fortunately the setup.exe hasn’t changed much, so it is still a matter of:

1. Copying the RTM ISO on a read-write filesystem
2. Unpack the Cumulative update of your choice
3. Amend the defaultsetup.ini file to indicate where the Cumulative Update files are to be found
4. Rebuild the ISO with a tool such as [OSCDIMG.EXE](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/oscdimg-command-line-options?view=windows-11). This tool can be found in the [Windows Assessment and Deployment kit](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install)

Here’s the code which can be also found on [GitHub](https://www.github.com/1Dimitri/sqlslip).

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="shell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">:: SQL Server 2019 Adding CU in a RTM Distribution and create ISOs

@echo off
if "%3"=="" goto :usage
if errorlevel 9009 echo OSCDIMG.EXE was not found, ISO will not be created, please add /noiso

setlocal
set mediasource=%1
set mediacopy=%~dpn3
if exist %mediacopy% ( echo deleting %mediacopy% && rd %mediacopy% /s /q) 
echo Copying %mediasource% (R/O) to %mediacopy%
robocopy %mediasource% %mediacopy% /E /COPY:DT
set tempcu=%mediacopy%\cu
for %%i in (%2\*.exe) do (echo Extracting %%i to %tempcu% && %%i /x:%tempcu% /q )

for %%i in (x64) do (
Echo creating/adding to defaultsetup.ini if necessary in %%i
if not exist %mediacopy%\%%i\defaultsetup.ini [OPTIONS]>%mediacopy%\%%i\defaultsetup.ini
attrib -r %mediacopy%\%%i\defaultsetup.ini
echo CUSOURCE=".\CU">>%mediacopy%\%%i\defaultsetup.ini
)
Echo creating ISO from %mediacopy% into %3
set label=%~n2
if /I "%4"=="/noiso" goto :noiso
if /I "%5"=="/noiso" goto :noiso

oscdimg -o -l%label% -u2 %mediacopy% %3
:noiso

if /I "%4"=="/nocleanup" goto :nocleanup
if /I "%5"=="/nocleanup" goto :nocleanup

rd %mediacopy% /s /q
goto :theend
:nocleanup
echo No cleanup made. Intermediate files in %mediacopy%
:theend
endlocal
goto :eof
:usage
echo %~n0 SQLSource-Directory CU-Directory TargetDirectory [/nocleanup] [/noiso] 
:: echo SQLSource-Directory will be modified, so make a copy of the RTM distribution before applying this batch!
echo e.g.
echo %~nx0 E:\ C:\CU14 D:\Temp\SQLSERVER2019CU14.ISO
echo.
echo E:\ is a read-only source for the RTM version of SQL Server (e.g. mounted ISO)
echo C:\CU14 is the location of the *untouched* KB exe of the CU 
echo    CU14 will be then the label of the ISO, so you may want to name the directory like SQL2019CU14 instead.
echo D:\temp\SQLSERVER2019CU14 side-by-side with the iso will be a temporary directory which will be erased afterwards
echo   /nocleanup will not remove  D:\temp\SQLSERVER2019CU14  
echo   /noiso builds the structure and not the ISO
echo. 
echo the build of the iso requires ISOCDIMG.EXE in the path
```