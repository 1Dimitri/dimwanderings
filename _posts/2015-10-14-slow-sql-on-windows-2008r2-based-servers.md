---
id: 374
title: 'Slow SQL on Windows 2008R2 based Servers'
date: '2015-10-14T19:40:06+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=374'
permalink: /2015/10/14/slow-sql-on-windows-2008r2-based-servers/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:57;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
tags:
    - hotfix
    - MAXDOP
    - performance
    - slow
    - workaround
---

## Sudden slow SQL symptom

If you have installed SQL Server on a Windows 2008 R2 OS, you may receive calls from your users complaining about slow SQL.

All your queries are becoming suddenly slow. THat’s a standard rant. But what is strange is it doesn’t affect user stored procedures only.

This includes standard procedures such as sp\_who, sp\_who2 or derivatives like the so-called [sp\_who3](http://sqlserverplanet.com/dba/a-better-sp_who2-using-dmvs-sp_who3),. All your SELECT, INSERT, UPDATE STATISTICs seem to be stuck in a SUSPENDed state without any open transaction or deadlock.

More strangely, command like SELECT \* on a table without index may work fine whereas SELECT COUNT(\*) start to take ages to complete. What’s going on?

By looking at the query plans, you may notice that the difference lies into the utilization of the aggregate operator, and that’s your clue. the Agggregate operator is used to make subqueries parallel.

Your parallelized queries run into trouble whereas simple non-parallel queries run fine.

[![Plan-Select-Clustered-Index-Scan](http://dimitri.janczak.net/wp-content/uploads/2015/10/Plan-Select-Clustered-Index-Scan.png)](http://dimitri.janczak.net/wp-content/uploads/2015/10/Plan-Select-Clustered-Index-Scan.png) [![Plan-Select-Count-Stream-Aggregate-Clustered-Index-Scan](http://dimitri.janczak.net/wp-content/uploads/2015/10/Plan-Select-Count-Stream-Aggregate-Clustered-Index-Scan.png)](http://dimitri.janczak.net/wp-content/uploads/2015/10/Plan-Select-Count-Stream-Aggregate-Clustered-Index-Scan.png)

## Solution

If you search for slow performance, you may hit [KB Article 2207548.](https://support.microsoft.com/en-us/kb/2207548) Both a hotfix and a workaround are suggested.

However, as long as the hotfix is not applied, applying the default ‘high performance’ power saving plan may still create issues. So better use the hotfix.

If you cannot reboot the server immediately, and as it a SQL Server it is likely going to be the case, you can still workaround this limitation by another mean. Just set the MAXDOP option to 1.

Remember that the MAXDOP is a server-wide setting. Potentially all applications should be checked against that setting.

This issue can also help you decide the priority for server migration. Replacing Windows 2008R2 by Windows 2012R2 is another way to solve the issue.

Note that the KB article is unclear about which CPU / BIOS are affected or why the symptom suddenly appears.