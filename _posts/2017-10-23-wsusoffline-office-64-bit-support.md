---
id: 777
title: 'WSUSOffline Office 64-bit support'
date: '2017-10-23T12:21:36+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=777'
permalink: /2017/10/23/wsusoffline-office-64-bit-support/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"778";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/10/WSUSOffline-Office-Tab.png
categories:
    - WSUSOffline
tags:
    - 64-bit
    - Office
    - patch
---

WSUSOffline Office 64-bit support is native but is not activated by default nor it is activable using the graphical interface.

[ WSUSOffline](http://www.wsusoffline.net/) is a tool that allow you tu update OS and Office products by using a share or an ISO image, for those who have to live in a world where servers and clients are not necessarily connected to the Internet or a WSUS Server. You prepare an image on a machine which has access to an Internet connection and this machine downloads the appropriate patches based on the monthly release of the well-known WSUSSCN2.CAB file.

If you click on one of the default check-boxes for the languages of Office, you will only get the 32-bit version of the patches (and the service packs if you have clicked that check-box too.)

[![WSUS Offline Office Tab](https://dimitri.janczak.net/wp-content/uploads/2017/10/WSUSOffline-Office-Tab.png)](https://dimitri.janczak.net/wp-content/uploads/2017/10/WSUSOffline-Office-Tab.png)

In order to get the 64-bit Office patches, you must run once the ‘AddOfficex64Support.cmd’ file you’ll find in the cmd subdirectory. You must append the 3-digit letter of the languages you want to have support for.

[![WSUS Offline cmd subdirectory](https://dimitri.janczak.net/wp-content/uploads/2017/10/WSUSOffline-cmd-subdirectory-contents.png)](https://dimitri.janczak.net/wp-content/uploads/2017/10/WSUSOffline-cmd-subdirectory-contents.png)Such a piece of code would be:

```
<pre class="lang:batch decode:true " title="Adding Office 64-bit support for WSUSOffline">AddOffice2010x64Support.cmd enu
```

With that operation, the lists of links will then be amended with the proper definitions and you do not need to redo this operation when you generate the new version the next version. In the same spirit, you would use RemoveOffice2010x64support.cmd to undo what you have done.

However if you’re using a Continuous Integration Server like [CruiseControl.Net](http://www.cruisecontrolnet.org/), [Jenkins](https://jenkins.io/) you may need to add this task for every build. Note also that despite the name of the command-line file, WSUSOffline Office 64-bit support is activated for all Office versions with that additional step.