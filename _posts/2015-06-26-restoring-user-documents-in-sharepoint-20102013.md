---
id: 226
title: 'Restoring user documents in Sharepoint 2010 / 2013'
date: '2015-06-26T20:26:17+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=226'
permalink: /2015/06/26/restoring-user-documents-in-sharepoint-20102013/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"225";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/06/SharePoint_2013_Logo.png
categories:
    - Sharepoint
tags:
    - backup
    - design
    - 'granular restore'
    - Import-SPWeb
    - operation
    - recover
    - restore
---

If you’re a Sharepoint architect, you have probably thought about the backup and restore strategy for the machines that make your farm: SQL Servers, Front-End Servers. However thinking about user cases may allow to tweak the overall strategy and reduced the perceived SLA by the end-user.

Let’s imagine the common of a user who requests to get the latest version or a previous one because he mistakenly has erased or simply overwritten the document he put so many efforts into.

There are basically three ways we can think about to reduce the time the IT support has to take to have the document back online.

1. Use Library version system, even your users say they wouldn’t need that feature. In addition to careful consideration about the storage, you may want to ask your users how often they work in a hurry. If they save the document ten times in a morning just before they need it for a meeting, you want to be sure that your major/minor version strategy does take that into account. One benefit is that they can self-restore if they have proper rights
2. Second level is the use of the recycle bin for erased documents. This doesn’t help if the document has been overwritten, but once again, it can save your IT Support calls for the first-level recycle bin. For the second-level, your administrator can quicker proceed with one click than multiple operations of database restore. Make your users and IT Support aware that deletion of a folder implies that they must restore the folder before restoring the item.
3. If you have regular SQL Server backups, you can use the ‘restore from an unattached database’ feature. Please make sure that your SQL Server infrastructure can support it. This means that 
    - either your SQL Server production has enough power (CPU, storage) to do that in parallel to its production duties
    - either you have a spare SQL Server at the same level (Version, Service Pack and CU) than your production one
    - The idea is then to 
        - restore the SQL Server backup. That’s where the site size matters.
        - go to the central administration
        - In backup and restore, select Recover from an unattached database and indicate the proper SQL Server name and database
        - Then to do the most granular possible export by targeting the site and the list you have to recover from. That’s where the list size matters.
        - Import this ‘export’ on the same site if your user doesn’t care about any data or on an alternate site if you’re concerned by one document only. Don’t forget that this step has nop GUI, you must use Import-SPWeb, and this cmdlet has the gotcha not to support relative paths, so always use some command like: Import-Web http://mytempsite -Path (Resolve-Path .\\myexport.cmp)

These operations, and in particular the last one, should also help you pinpoint to your users the importance to have lists with a limited number of items if they want to have quick restores and good performance.