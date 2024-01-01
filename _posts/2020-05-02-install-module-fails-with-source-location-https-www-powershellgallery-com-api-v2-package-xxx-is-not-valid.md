---
id: 1004
title: 'Install-Module fails with Source Location https://www.powershellgallery.com/api/v2/package/xxx is not valid'
date: '2020-05-02T20:57:38+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=1004'
permalink: /2020/05/02/install-module-fails-with-source-location-https-www-powershellgallery-com-api-v2-package-xxx-is-not-valid/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1008;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2020/04/Install-Module-Powershell-Source-Location-Is-Not-Valid.png
categories:
    - 'troubleshooting tips'
tags:
    - download
    - https
    - powershell
    - 'powershell module'
    - TLS
---

The use of Install-Module and Update-Module has become a routine task for many Powershell administrators these days.

Lately if you applied those commands, you may have received warnings and errors even from well-known sources. Messages include:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">WARNING: Source Location 'https://www.powershellgallery.com/api/v2/package/Name/Version' is not valid.
PackageManagement\Install-Package : Package 'Name' failed to download.
At C:\Program Files\WindowsPowerShell\Modules\PowerShellGet\1.0.0.1\PSModule.psm1:1772 char:21
+ ...          $null = PackageManagement\Install-Package @PSBoundParameters
+                      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ResourceUnavailable: (C:\Users\...r\Name.nupkg:String) [Install-Package], Excep
   tion
    + FullyQualifiedErrorId : PackageFailedInstallOrDownload,Microsoft.PowerShell.PackageManagement.Cmdlets.InstallPackage
```

If you look at the resource or the web site, the package stills exists and can be manually downloaded with a recent browser.

In fact, like many https resource nowadays, the solution lies in the https support in terms of cryptograthy algorithms either on the client or on the server.

Here you will have to add the support for TLS 1.2 for the package to be downloaded, with the following command at the powershell prompt:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

And then the operations will resume as usual… Until the next mismatch with the alignment of TLS Support between servers and clients !

<figure class="wp-block-image size-large">![](https://dimitri.janczak.net/wp-content/uploads/2020/04/Install-Module-Powershell-Source-Location-Is-Not-Valid.png)</figure>