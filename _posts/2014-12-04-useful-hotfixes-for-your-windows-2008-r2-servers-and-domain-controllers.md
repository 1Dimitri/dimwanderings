---
id: 68
title: 'Useful hotfixes for your Windows 2008 R2 Servers and Domain Controllers'
date: '2014-12-04T19:47:05+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=68'
permalink: /2014/12/04/useful-hotfixes-for-your-windows-2008-r2-servers-and-domain-controllers/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"70";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/12/windows-update-3d.jpg
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - 'Windows 2008 R2'
---

In addition to the monthly security patches from Microsoft, some hotfixes are worth to apply on Windows 2008 R2 servers and in particular on Domain Controllers.

First of all, there’s the [enterprise rollup package](http://support.microsoft.com/kb/2775511) which is available in the Microsoft Catalog but is not available to Update Services by default. It is worthwhile to mention that revisions of this article do mention regression issues and after this package, three additional msu files must be installed.

Secondly, for any DFS-R machine, including the domain controllers as far as the SYSVOL replication is concerned, you’d better align your policy of auto recovery in case of corruption with the new recommended default Microsoft applies starting with WIndows 2012: this implies to install [this hotfix](http://support.microsoft.com/kb/2884176) and to create a registry key. Be careful that for once the key StopReplicationOnAutoRecovery must be put to 0. You may want to create a group policy preference and link it to your Domain Controller OU.

Note that for DFS related hotfixes, for both namespaces and replication, there’s a [knowledge base article](http://support.microsoft.com/kb/968429) which is not to be missed to know what’s the latest recommendation.

Your Domain Controller is probably also a DNS Server. In this case, [this hotfix](http://support.microsoft.com/kb/2905363) may be useful as it contains one of the latest version of dns.exe. Speaking of DNS, you may have heard of [this problem](http://support.microsoft.com/kb/968372) of top-level domain resolution. It is not Microsoft implementation related, but a Group Policy object linked to your Domain Controllers may also help.

As far as the Windows ACtive Directory itsdelf is concerned,[ this hotfix](http://support2.microsoft.com/kb/2862304) will help you get more performance and [this one](http://support2.microsoft.com/kb/2800945) will help you do troubleshooting.