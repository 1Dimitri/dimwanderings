---
id: 921
title: 'Your SQL Server installation is either corrupt or has been tampered with'
date: '2019-02-15T15:12:27+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=921'
permalink: /2019/02/15/your-sql-server-installation-is-either-corrupt-or-has-been-tampered-with/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:57;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
    - 'troubleshooting tips'
tags:
    - sqlservr
---

When receiving this error message: ”  
`Error : Your SQL Server installation is either corrupt or has been tampered with (Error getting instance ID from name.). Please uninstall then re-run setup to correct this problem`, it is likely you are trying to start SQL Server as a regular binary or as a service without any command line parameter.  
Contrary to [this troubleshooting post](<http://Error : Your SQL Server installation is either corrupt or has been tampered with (Error getting instance ID from name.). Please uninstall then re-run setup to correct this problem>), this may happen also when you have a single SQL Server instance on your machine.  
The trick is to always use the -s switch to specify the instance name.  
Please note that in this context, the instance name does not include the server name. Therefore a MYSERVER\\MYSQLSERVERINSTANCE standard notation should be translated into:  
`sqlservr.exe -s MYSQLSERVERINSTANCE`