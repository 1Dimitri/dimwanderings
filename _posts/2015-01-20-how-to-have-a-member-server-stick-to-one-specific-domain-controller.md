---
id: 100
title: 'How to have a member server stick to one specific domain controller?'
date: '2015-01-20T20:17:46+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=100'
permalink: /2015/01/20/how-to-have-a-member-server-stick-to-one-specific-domain-controller/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"101";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2015/01/glu_stick.jpg
categories:
    - 'troubleshooting tips'
tags:
    - 'Active Directory'
    - authentication
---

In case of troubleshooting or temporary issue, you may want to be sure that a given member server always authenticate against one specific domain controller.

For this, you may want to combine two tools:

1. The “time-to-live” (TTL) value for the validity of the discovered domain controller
2. The ability to specifically target a given domain controller when checking the secure chanel

The first point is addressed by using the ForceRediscoveryInterval registry key introduced in Windows 2008 (and back ported to Windows 2003 in a hotfix should legacy servers live after july 2015 at your premises). It is documented in detail in the incorrectly titled article [KB 939252: The domain controller locator cannot find an appropriate domain controller on a computer that is running Windows XP or Windows Server 2003](http://support.microsoft.com/kb/939252/en-us).  
The `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters\ForceRediscoveryInterval` DWORD value should be set to `0xffffffff` for (almost) eternity to happen.

The second part is achieved using the regular

```
<pre class="lang:batch decode:true " title="nltest to target a specific Domain Controller">nltest /server:The_DC_I_Want /sc_query:MyDomain
```

Please note that this trick doesn’t allow you to make this setting persist across reboots.