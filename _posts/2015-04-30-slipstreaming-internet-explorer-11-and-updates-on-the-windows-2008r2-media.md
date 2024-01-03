---
id: 131
title: 'Slipstreaming Internet Explorer 11 and updates on the Windows 2008R2 media'
date: '2015-04-30T20:12:55+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=131'
permalink: /2015/04/30/slipstreaming-internet-explorer-11-and-updates-on-the-windows-2008r2-media/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"70";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/12/windows-update-3d.jpg
categories:
    - 'Windows OS'
tags:
    - installation
    - slipstream
---

There has been a while since Microsoft did not publish any service pack for Windows 2008 R2, so if you are involved in the business of regularly installing Windows Operating Systems and do not want to install hundred of updates by hand, you have to create your own slipstreamed CD/DVD/share.

That’s in particular true if you want to avoid wasting time installing the Windows 2008R2, then all the patches, then Internet Explorer 11 and then the Internet Explorer 11 related patches.

Helped with Microsoft documentation, you naively figure out that integrating all your WSUS or manually downloaded patches with the help of the dism tool would suffice; you’re wrong! If you do so you resuting WIM file is not working. You’ll have to order that mess!

A good starting point is KB 2847882, which gives a list of prerequisites for Internet Explorer 11, namely at the time of this writing:

- KB2670838
- KB2834140
- KB2639308
- KB2533623
- KB2731771
- KB2729094
- KB2786081
- KB2888049
- KB2882822

But this is not sufficient, some patches cannot be directly integrated either because they require that the OS be running or that they are depending on .Net Framework 4, which is not installed by default in Windows 2008 R2. Those patches are as of now:

- KB2496898
- KB2533552
- KB2604521
- KB2726535
- KB2819745
- KB2506143