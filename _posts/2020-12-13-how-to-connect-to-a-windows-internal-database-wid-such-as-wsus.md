---
id: 1039
title: 'How to connect to a Windows Internal Database WID such as WSUS'
date: '2020-12-13T11:37:44+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=1039'
permalink: /2020/12/13/how-to-connect-to-a-windows-internal-database-wid-such-as-wsus/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1042;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2020/12/WSUS-Windows-Internal-Database-Connection-String-Windows-2012R2-2016.png
categories:
    - troubleshooting
    - 'Windows Server 2016'
    - 'WSUS (Windows Server Update Services)'
tags:
    - 'SQL Server'
    - WID
    - 'Windows Internal Database'
    - WSUS
---

The Windows Internal Database is nothing more than an embedded lite SQL Server instance running under specific credentials and with a few protocols enabled.

Therefore in order to connect, you can use any SQL Server client tool: SQL Server Management Studio, SQLCMD, …

<figure class="wp-block-image size-large">![](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-Windows-Internal-Database-Connection-String-Windows-2012R2-2016.png)</figure>Starting with Windows Server 2012, the named pipe to access is `\\.\pipe\MICROSOFT##WID\tsql\query`. For older systems, the pipe is different and is named `\\.\pipe\MSSQL$MICROSOFT##SSEE\sql\query`.

From this we can already sense that this is some kind of named instance now called MICROSOFT##WID and MICROSOFT##SSEE in the past.

Please note that the tools must be run with elevated privileges, otherwise you’ll get an access denied message.

<figure class="wp-block-image size-large">![Windows Internal Database Login failed](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-Windows-Internal-Database-Connection-Login-Failed.png)</figure>Once logged in we can see that Windows 2016 Windows Internal database feature hosts a SQL Server 2014 RTM like version.

<figure class="wp-block-image size-large">![WSUS SUSDB in SSMS](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-Windows-Internal-Database-From-SSMS-18.png)</figure>The SUSDB is a regular user database created by the WSUS feature. If we look into logins, they can be split into 3 groups:

- the built-in logins you’ll find in every SQL Server, the ##XXXX certificate ones, sa, etc.
- a set of logins for the WID feature, which get the sysadmin role. The NT Service\\WIDWriter and the NT Service\\MSSQL$MICROSOFT##WID will again make you think about a named instance MICROSOFT#WID,
- a group here created by the client application (MACHINE\\WSUS administrators)

<figure class="wp-block-image size-large">![SQL Login in Windows Internal Database WSUS](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-WID-Logins.png)</figure><figure class="wp-block-image size-large">![NT SERVICE\MSSQL$MICROSOFT##WID](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-WID-Login-WID-Sysadmin.png)</figure>If we dig into the characteristics of the SUSDB database, we see a regular user database for which some user roles are defined and attached to the specific logins we mentioned earlier.

<figure class="wp-block-image size-large">![WSUS SUSDB Views in SSMS](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-SUS-DB-Views-From-SSMS-18.png)</figure><figure class="wp-block-image size-large">![SUSDB Database security users & roles](https://dimitri.janczak.net/wp-content/uploads/2020/12/WSUS-SUSDB-Database-Security.png)</figure>Needless to say that you can issue backups of the SUSDB as you would for any regular database, although officially Microsoft maintains there is no need to deploy any maintenance solution.

You may want to note that this SQL Server “Edition” behaves like the Express Edition under some circumstances, e.g. contrary to the Standard, Enterprise or Web Edition, it doesn’t eat as much as RAM as it can.