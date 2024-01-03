---
id: 55
title: 'Slipstreaming SQL Server 2008 / 2008 R2 service packs'
date: '2014-10-12T22:27:25+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=55'
permalink: /2014/10/12/slipstreaming-sql-server-2008-2008-r2-service-packs/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"57";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
tags:
    - CU
    - 'service pack'
    - slipstream
    - snippet
---

Here’s a little batch script to automatically integrate a service pack into your SQL Server RTM files.  
It has been tested with SQL Server 2008 and SQL Server 2008 R2.

Please note that the latest service packs ([Service Pack 4 for SQL Server 2008](http://go.microsoft.com/fwlink/?LinkID=512814 "SQL Server 2008 Service Pack 4 Download Page") and [Service Pack 3 for SQL Server 2008 R2](http://go.microsoft.com/fwlink/?LinkID=512818 "SQL Server 2008 R2 Service Pack 3 Download Page")) do not include Itanium releases and will throw up some errors.

```
<pre class="lang:batch decode:true " title="SQL Server Service Pack integration">:: SQL Server 2008 Adding Service Packs in %2 to a RTM distribution in %1
:: DJ - 13.07.10 - 1.0 - Tested w/ SP1
:: DJ - 04.01.11 - 1.1 - Tested w/ SP2
@echo off
if "%2"=="" goto :usage
setlocal
set mediacopy=%1
set temppcu=%mediacopy%\pcu
for %%i in (%2\*.exe) do (echo Extracting %%i to %temppcu% && %%i /x:%temppcu% /q )
for %%i in (setup.exe setup.rll) do (Echo copying %%i to %mediacopy% && robocopy %temppcu% %mediacopy% %%i)
for %%i in (x86 x64 ia64) do (Echo copying %%i to %mediacopy% && robocopy %temppcu%\%%i %mediacopy%\%%i /XF Microsoft.SQL.Chainer.PackageData.Dll)
for %%i in (x86 x64 ia64) do (
Echo creating/adding to defaultsetup.ini if necessary in %%i
if not exist %mediacopy%\%%i\defaultsetup.ini cmd /U /C echo [SQLSERVER2008]>%mediacopy%\%%i\defaultsetup.ini
cmd /U /C echo PCUSOURCE=".\PCU">>%mediacopy%\%%i\defaultsetup.ini
)
endlocal
goto :eof
:usage
echo %~n0 SQLSource-Directory ServicePacks-Directory
echo SQLSource-Directory will be modified, so make a copy of the RTM distribution before applying this batch!

```