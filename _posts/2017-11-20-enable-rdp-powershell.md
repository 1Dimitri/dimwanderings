---
id: 793
title: 'Enable RDP with Powershell'
date: '2017-11-20T17:13:42+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=793'
permalink: /2017/11/20/enable-rdp-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"794";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/11/RDP-Properties.png
categories:
    - 'Windows OS'
tags:
    - powershell
    - RDP
    - unattended
---

Although the Remote Management is enabled by default starting with Windows Server 2012, the RDP access isn’t. To enable RDP with Powershell, you have two steps to perform:

1. Enable the RDP Access
2. Enable the inbound firewall rules

Here’s the code:

```
# 1. Enable Remote Desktop
(Get-WmiObject Win32_TerminalServiceSetting -Namespace root\cimv2\TerminalServices).SetAllowTsConnections(1,1) 
(Get-WmiObject -Class "Win32_TSGeneralSetting" -Namespace root\cimv2\TerminalServices -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0) 

# 2. Enable the firewall rule
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

The second line allows you to specify if you want to use NLA or not.