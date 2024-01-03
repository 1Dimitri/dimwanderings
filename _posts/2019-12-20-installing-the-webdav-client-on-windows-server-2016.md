---
id: 991
title: 'Installing the WebDav client on Windows Server 2016'
date: '2019-12-20T13:30:40+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=991'
permalink: /2019/12/20/installing-the-webdav-client-on-windows-server-2016/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:992;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2019/12/WebRedirector-2016-Set-Service.png
categories:
    - 'Windows Server 2016'
tags:
    - 'Add Roles and Features'
    - WebDAV
    - workstation
---

As many remote storage solutions now provide a [WebDav](https://en.wikipedia.org/wiki/WebDAV) access (from Sharepoint to Nextcloud), having the ability to natively upload and download files from a WebDav Server on Windows Server 2016 may be useful.

As opposed to the previous versions, you do not need to install the Desktop Experience feature to make this available on Windows Server 2016. However you’ll need a reboot and to tweak some services.

Start by installing the feature called “WebDav-Redirector”.

```
<pre class="wp-block-code">```
Install-WindowsFeature WebDav-Redirector -Restart
```
```

The reboot is mandatory so if you don’t specify the restart parameter, you’ll get this error :

<figure class="wp-block-image">![](https://dimitri.janczak.net/wp-content/uploads/2019/12/WebRedirector-2016-Install-Feature.png)</figure>In fact, the reboot is needed as the package is made of a service and a driver, and they’re not installed unless you reboot:

<figure class="wp-block-image">![](https://dimitri.janczak.net/wp-content/uploads/2019/12/WebRedirector-2016-Services-Not-Yet-Installed.png)</figure>For some unknown reason, the services are set to be manual triggered. If you want to have them for any task, you must set them as ‘automatic’:

<figure class="wp-block-image">![](https://dimitri.janczak.net/wp-content/uploads/2019/12/WebRedirector-2016-Set-Service.png)</figure>```
<pre class="wp-block-code">```
@('WebClient','MRxDAV') | Foreach-Object { Set-SErvice $_ -StartupType Automatic }
```
```

Then you’ll be able to issue commands such as :

```
<pre class="wp-block-code">```
net use * \\live.sysinternals.com
```
```