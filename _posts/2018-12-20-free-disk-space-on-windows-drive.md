---
id: 915
title: 'Free Disk space on Windows drive'
date: '2018-12-20T15:00:52+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=915'
permalink: /2018/12/20/free-disk-space-on-windows-drive/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:916;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/12/dism-cleanup-example.png
categories:
    - 'troubleshooting tips'
tags:
    - cleanup
    - dism
    - maintenance
    - space
    - winsxs
---

Starting with Windows 2012 (R2), it is no longer necessary to install the desktop experience to remove the different file versions left update after update. The following command can be used:

```
<pre class="lang:batch decode:true " title="Regaining space on Windows System drive">dism.exe /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```

Options used here are:

- StartComponentCleanup: removes superseded and unused system files from a system
- ResetBase: all superseded versions of every component in the component store is removed.

The resetbase can be quite interesting, On a 3+ years old Windows 2012R2 the lack of ResetBase allows to gain 4GB, its use averaged for 20GB free space.

To get a report view before doing anything damageable, you can run:

```
<pre class="lang:batch decode:true " title="Reporting  Space used by WInSxS folder">dism.exe /Online /Cleanup-Image /AnalyzeComponentStore
```

The command has been succesfully tested on Windows 2012R2 and Windows Server 2016.