---
id: 529
title: 'Retrieving SID in Powershell'
date: '2016-11-09T09:29:44+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=529'
permalink: /2016/11/09/retrieving-sid-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:530;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - code
tags:
    - account
    - SID
---

Retrieving a SID can be usually done with [psgetsid](https://technet.microsoft.com/en-us/sysinternals/bb897417.aspx) or [whoami ](https://technet.microsoft.com/en-us/library/cc771299(v=ws.11).aspx)/all

Here’s a quick snippet to help get you the Secure Identifier (SID) of a local or domain Windows Account within Powershell

```
<pre class="lang:ps decode:true " title="Getting a SID">$u = New-Object System.Security.Principal.NTAccount('username')
$SID = $u.Translate([System.Security.Principal.SecurityIdentifier])
$SID.Value

```

And here’s an example:

[![Retrieving SID in Powershell](https://dimitri.janczak.net/wp-content/uploads/2016/11/SID-Administrator-Powershell.png)](https://dimitri.janczak.net/wp-content/uploads/2016/11/SID-Administrator-Powershell.png)

In case you need to check the well-known SID, they are listed in [this article](https://support.microsoft.com/en-us/kb/243330).