---
id: 941
title: '‘MEDIALAYOUT’ in SQL Server ConfigurationFile.Ini'
date: '2019-04-04T16:21:47+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=941'
permalink: /2019/04/04/medialayout-in-sql-server-configurationfile-ini/
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
    - troubleshooting
    - unattended
---

While trying to install new instance of SQL Server using command line and the ConfigurationFile.ini unattended answer file, you may receive the following error message : MEDIALAYOUT in SQL Server ConfigurationFile.Ini

```
<pre class="wp-block-code">```
The specified value for setting ‘MEDIALAYOUT’ is invalid. The expected values are:
None
Core
Advanced
Full
Error code 0x84B40001. 
```
```

Don’t waste your time searching in different documentations about the allowed keywords for the configuration file.

This error means you forgot to put slashes in front of every argument you used on the command line after setup.exe, e.g.

```
<pre class="wp-block-code">```
setup.exe configurationfile 
```
```

Just add all the slashes back, and voilà!