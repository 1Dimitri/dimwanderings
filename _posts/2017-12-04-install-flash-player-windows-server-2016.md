---
id: 797
title: 'Install Flash Player on Windows Server 2016'
date: '2017-12-04T10:40:47+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=797'
permalink: /2017/12/04/install-flash-player-windows-server-2016/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"800";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/12/Adobe_Flash_Player_v10_icon.png
categories:
    - troubleshooting
    - 'Windows Server 2016'
tags:
    - 'Desktop Experience'
    - Flash
    - package
    - vmware
    - 'vSphere Client'
    - 'Windows Server 2016'
---

When you’re dealing with some legacy software, such as the[ vSphere web client version 5.x](https://blogs.vmware.com/vsphere/2016/05/goodbye-vsphere-client-for-windows-c-hello-html5.html) and you’re accessing it from a Windows Server OS, You may need to install Flash Player on Windows Server 2016 for Internet Explorer. If you run a search through Google, Bing, or Qwant, you may find various entries suggesting:

- to enable Remote Desktop Host role
- to enable the Desktop experience role
- [Windows 10 only solution](https://helpx.adobe.com/flash-player/kb/flash-player-issues-windows-10-ie.html)

Fortunately, there’s no need for that. You just to have to install the proper package. Unfortunately it is not displayed as a stand-alone feature in the Server Manager wizard. You’ll have to revert to a bit a Powershell (for laziness) and dism to install package online. Here is the code:

```
<pre class="lang:ps decode:true " title="Installing the Flash Player package on Windows Server 2016">$flash = gci (Join-Path $Env:WinDir 'Servicing\Packages')  -Filter 'Adobe-Flash-For-Windows-Package*.mum'
dism /online /add-package /packagepath:$($flash.FullName)
```

In the code you may note the use of the Fullname property. Don’t forget that the objects returned by Get-ChildItem are FileInfo and that the default string of those objects are the filename not the full path name. Therefore, if you were to execute this command:

```
<pre class="lang:default decode:true " title="Failing install package">dism /online /add-package /packagepath:$flash
```

you’d likely to get:

```
<pre class="lang:default decode:true " title="Dism Error package not found">Deployment Image Servicing and Management tool
Version: 10.0.14393.0

Image Version: 10.0.14393.0

An error occurred trying to open - Adobe-Flash-For-Windows-Package~31bf3856ad364e35~amd64~~10.0.14393.0.mum Error: 0x80070003
An error occurred trying to open - C:\Windows\system32\Adobe-Flash-For-Windows-Package~31bf3856ad364e35~amd64~~10.0.14393.0.mum Error: 0x80070003

Error: 3

An error occurred trying to open - C:\Windows\system32\Adobe-Flash-For-Windows-Package~31bf3856ad364e35~amd64~~10.0.14393.0.mum Error: 0x80070003
```

The correct execution path is:

```
<pre class="lang:default decode:true" title="Successful dism package execution">Version: 10.0.14393.0

Image Version: 10.0.14393.0

Processing 1 of 1 - Adding package Adobe-Flash-For-Windows-Package~31bf3856ad364e35~amd64~~10.0.14393.0
[==========================100.0%==========================]
The operation completed successfully.
```

[![](https://dimitri.janczak.net/wp-content/uploads/2017/12/get-flash-package-location.png)](https://dimitri.janczak.net/wp-content/uploads/2017/12/get-flash-package-location.png)  
[![](https://dimitri.janczak.net/wp-content/uploads/2017/12/dism-install-flash-package.png)](https://dimitri.janczak.net/wp-content/uploads/2017/12/dism-install-flash-package.png)  
Once you’ve done that, you just need to close your Internet Explorer Window and re-open the site with the embedded flash contents.

Please note that once enabled, you may still have issues with [Flash and the vSphere client](https://www.virtuallyghetto.com/2017/10/shockwave-flash-has-crashed-workaround-for-vsphere-web-flash-client.html), but that’s linked to Flash, not to the handling of Flash Player by Windows and Internet Explorer anymore.