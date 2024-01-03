---
id: 356
title: 'Find the RDS Licensing servers in your domain'
date: '2015-08-21T20:44:58+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=356'
permalink: /2015/08/21/find-the-rds-licensing-servers-in-your-domain/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"358";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/08/Remote-Desktop-Services-Roles.png
categories:
    - 'Powershell Engine'
    - 'Remote Desktop Services'
tags:
    - 'RDS Licensing'
    - SCP
    - 'Service Connection Point'
---

During good old Windows 2003 times, the Terminal Licensing Servers were published in the Active Directory using the Sites object and the entry *TS-Enterprise-License-Server* as [explained here](http://blogs.technet.com/b/asiasupp/archive/2007/03/27/manually-publishing-and-un-publishing-terminal-server-license-servers.aspx).

Under WIndows 2008R2, the RDS Licensing role service registers a service connection point; however few documents tells you where to find this SCP in your AD: it is a CN called ‘TermServLicensing’ under the CN of the computer account. For example, if your server rdslic.mydomain.local is located in a top-level OU called ‘Servers’, the object will be called:

```
<pre class="">CN=TermServLicensing,CN=rdslic,OU=Servers,DC=mydomain,DC=local
```

To find out all registered SCP, you can issue one of the following commands:

```
<pre class="lang:batch decode:true" title="finding RDS Licensing servers using cmd">dsquery * -filter "(&(CN=TermServLicensing)(objectClass=serviceConnectionPoint))"
```

or

```
<pre class="lang:default decode:true " title="finding RDS Licensing servers using Powershell"># one of the following:
Get-ADObject -LDAPFilter "(&(CN=TermServLicensing)(objectClass=serviceConnectionPoint))"
Get-ADObject -Filter {objectClass -eq 'serviceConnectionPoint' -and Name -eq 'TermServLicensing'}
```