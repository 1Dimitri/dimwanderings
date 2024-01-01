---
id: 342
title: 'Registering a DLL assembly in the GAC using Powershell'
date: '2015-08-14T12:30:27+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=342'
permalink: /2015/08/14/registering-a-dll-assembly-in-the-gac-using-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";b:0;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - 'Powershell Engine'
tags:
    - Assembly
    - GAC
---

If you are managing Sharepoint or Active Directory Federation Services, you sometimes have to register DLLs in the GAC without your supplier providing you a nice MSI installer to do so.

Once solution would be use [gacutil](https://msdn.microsoft.com/en-us/library/ex0ss12c%28v=vs.110%29.aspx) but it is [now](http://stackoverflow.com/questions/2182316/how-to-register-a-net-dll-in-gac) a .Net Framework Utility, and you may not want to copy it on your servers. To accomplish the same thing using Powershell solely, you may run these lines:

```
<pre class="lang:ps decode:true " title="Assembly registration in GAC in Powershell">[System.Reflection.Assembly]::Load("System.EnterpriseServices, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
$publisher = New-Object System.EnterpriseServices.Internal.Publish
$publisher.GacInstall("C:\path\MydllToRegister.dll")
```

Don not forget that you can retrieve many properties of the assembly, such as the public token, just by loading it:

```
<pre class="lang:ps decode:true " title="Retrieving Properties of an Assembly">([system.reflection.assembly]::loadfile("C:\path\MydllToRegister.dll")).FullName
```