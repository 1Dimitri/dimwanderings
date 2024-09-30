---
id: 111
title: 'Special Folders in Powershell'
date: '2015-02-08T15:20:26+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=111'
permalink: /2015/02/08/special-folders-in-powershell/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"18";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/08/powershell-logo-e1410861403696.jpg
categories:
    - 'Powershell Engine'
tags:
    - 'common folder'
    - desktop
    - documents
    - explorer
    - folders
    - snippet
    - 'special folder'
---

A good start point is the environment variables special powershell drive, aka Env:. However all folders are not available through this mechanism.

You may want to build a hash table to retrieve them all and optionally add a drive to make your code nicer to read. The code is based on the GetNames static method of an enum to retrieve the various names, knowing that the enumeration we are interested in are Environment.SpecialFolder.

So here we go:

```
$SpecialFolders = @{}
$names = [Environment+SpecialFolder]::GetNames([Environment+SpecialFolder])
foreach($name in $names)
{
  if($path = [Environment]::GetFolderPath($name)){
    $SpecialFolders[$name] = $path
    # New-PSDrive -Name $name -PSProvider FileSystem -Root $path
  }
}
```