---
id: 533
title: 'Prefer IPv4 over IPv6 on Windows'
date: '2016-11-15T14:07:30+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=533'
permalink: /2016/11/15/prefer-ipv4-ipv6-windows/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:534;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/11/Windows-Default-IPv6-prefix-policy.png
categories:
    - 'Windows OS'
tags:
    - IPv4
    - IPv6
    - 'prefix policy'
    - TCP/IP
---

When trying to get rid of IPv6 for good or mostly bad read reasons, most searches on the Internet will link to [KB article 929852](https://support.microsoft.com/en-us/kb/929852) when it comes to prefer IPv4 over IPv6.

Basically, you must update under `HKLM\System\currentcontrolset\services\tcpip6\parameters`, the `DisabledComponents` key and use a value of 20.

However this method has two drawbacks:

- it requires you to reboot the computer
- there is no built-in GPO to do this, however [this post](http://social.technet.microsoft.com/wiki/contents/articles/5927.how-to-disable-ipv6-through-group-policy.aspx) has the admx files to do this.

An alternate method is to use the IPv6 prefix policies. They define which IPv6 prefixes are used preferably when sending packets. There you can choose if your settings are valid until the next reboot or are persistent across them.

On a standard Windows 2008R2, the prefix policy is as follows:

[![Windows Default IPv6 Prefix Policy](https://dimitri.janczak.net/wp-content/uploads/2016/11/Windows-Default-IPv6-prefix-policy.png)](https://dimitri.janczak.net/wp-content/uploads/2016/11/Windows-Default-IPv6-prefix-policy.png)Here the IPv6 addresses are preferred as ::/0 has a precedence of 40, just after the localhost ::1 entry. As you see at any time you can get the current state with `netsh interface ipv6 show prefixpolicies`.

To have IPv4 favored over IPv6, you must enter a command like `netsh interface ipv6 set prefixpolicy ::ffff:0:0/96 46 4` which will make IPv4 addresses have a precedence of 46. By default the entry is made persistent across reboots as opposed to good old route utility under IPv4.

By the way, if you’re lost in IPv6, the multiple RFCs which supersede each other, how IPv4 addresses are written in IPv6 language, there is a [nice cheat sheet here ](http://www.roesen.org/files/ipv6_cheat_sheet.pdf)by Jens Roesen.