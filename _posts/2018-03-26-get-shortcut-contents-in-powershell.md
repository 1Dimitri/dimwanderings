---
id: 840
title: 'Get Shortcut contents in Powershell'
date: '2018-03-26T12:02:35+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=840'
permalink: /2018/03/26/get-shortcut-contents-in-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"843";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/03/shortcut-explorer.png
categories:
    - 'Explorer Shell'
tags:
    - link
    - lnk
    - powershell
    - shell
    - shortcut
    - wscript
---

Some applications create a lot of shortcuts and sometimes you wonder what differences between them are. Or they are created at a locatoin you don’t find practical and want to create them elsewhere. You could click on each shortcut and copy paste the information but so let’s see how to get shortcut contents in Powershell so we can automate that.

```
<pre class="lang:default decode:true" title=".lnk Shortcut manipulation in Powershell">$sh = New-Object -ComObject WScript.Shell
$sm = [Environment]::GetFolderPath('CommonPrograms')
$link = (gci $sm -File -rec)[0]
$lnk = $sh.CreateShortcut($link.FullName)
$lnk.TargetPath
$lnk.Arguments
```

Manipulating the shortcuts is part of the shell responsibilities, so we will need to instantiate a [Wscript.Shell](https://msdn.microsoft.com/en-us/library/aew9yb99(v=vs.84).aspx) COM Object. In this example, we are retrieving the first shortcut found in the Start Menu of All users

Then you will need to point at the .lnk file and use the [CreateShortcut](https://msdn.microsoft.com/en-us/library/xsy6k3ys(v=vs.84).aspx) method. Despite its name the CreateShortcut method is not meant to create the lnk file but rather create a representation of the shortcut as an object. You get then various properties of a [WshShortcut](https://msdn.microsoft.com/en-us/library/xk6kst2k(v=vs.84).aspx) object whose most interesting ones are the TargetPath and the Arguments values.

A complete example here is the output of the different shortcuts created by Visual Studio 2017 for the developer tool prompts.

```
<pre class="lang:ps decode:true " title="Getting shortcuts of Visual Studio 2017 Developer Tools in Powershell">$sh = New-Object -ComObject WScript.Shell
$sm = [Environment]::GetFolderPath('CommonPrograms')
$vstools = [io.path]::Combine($sm,'Visual Studio 2017','Visual Studio Tools','VC')
Get-ChildItem -Path $vstools -File | ForEach-Object {
      $lnk = $sh.CreateShortcut($_.FullName)
      "$($_) -> $($lnk.TargetPath) $($lnk.arguments)" 
}
```

If you were to create or update a lnk shortcut file on Windows, you would still use CreateShortcut to manipulate the object, set up the properties and then you would zuse the Save() method to record the change into a newly create or existing .lnk file