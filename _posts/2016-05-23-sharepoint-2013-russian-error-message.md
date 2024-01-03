---
id: 436
title: 'Sharepoint russian error'
date: '2016-05-23T15:50:05+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=436'
permalink: /2016/05/23/sharepoint-2013-russian-error-message/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:439;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - Sharepoint
tags:
    - installation
    - msi
    - troubleshooting
---

Funny surprise today as I was repairing a Sharepoint 2013 installation.

Instead of insulting me in proper plain english, the server started to output some russian sentences as shown in the picture below:

[![Sharepoint 2013 Russian Error Message](http://dimitri.janczak.net/wp-content/uploads/2016/05/Sharepoint-2013-Russian-Error-Message.png)](http://dimitri.janczak.net/wp-content/uploads/2016/05/Sharepoint-2013-Russian-Error-Message.png)

Google Translate was of no help, but looking at the[ MSI Error code](https://msdn.microsoft.com/en-us/library/windows/desktop/aa372835%28v=vs.85%29.aspx) revealed the issue: the ISO image that was mounted during installation was no longer present.

Mounting the ISO image help the message to vanish.