---
id: 565
title: 'IPAM Server needs SQL Server Enterprise Edition'
date: '2016-12-21T13:18:19+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=565'
permalink: /2016/12/21/ipam-server-needs-sql-server-enterprise-edition/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:564;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - IPAM
tags:
    - installation
    - 'SQL Server'
    - troubleshooting
---

Although this is marked nowhere in the [installation documentation](https://technet.microsoft.com/en-us/library/jj878325(v=ws.11).aspx), but only in one blog post about moving[ IPAM database between servers or instances](https://blogs.technet.microsoft.com/yagmurs/2014/07/31/moving-ipam-database-from-windows-internal-database-wid-to-sql-server-located-on-the-same-server/), please be careful when designing your IPAM solution. As of today, in Windows 2012R2, your database for an IPAM server can only be:

- Windows Internal Database (Windows 2012 and WIndows 2012R2)
- SQL Server Enterprise Edition (Windows 2012R2)

SQL Server Express Edition is not supported which may not surprise you, but the SQL Server Standard Edition isn’t also.

If you try to provision your IPAM Server with a non-supported edition, you’ll get one of the following error messages:[![IPAM SQL Server Enterprise eroor message](https://dimitri.janczak.net/wp-content/uploads/2016/12/Ipam-Server-Provisioning-Needs-SQL-Server-Enterprise.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/Ipam-Server-Provisioning-Needs-SQL-Server-Enterprise.png)

- “IPAM supports only Enterprise Edition of SQL server but Express Edition of SQL Server is installed on %computername%”
- “IPAM supports only Enterprise Edition of SQL server but Standard Edition of SQL Server is installed on %computername%”

Reasons other than commercial limitations are unknown since it aldso works with Windows Internal Database