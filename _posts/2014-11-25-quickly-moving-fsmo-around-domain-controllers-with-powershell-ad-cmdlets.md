---
id: 49
title: 'Quickly moving FSMO around Domain Controllers with Powershell AD Cmdlets'
date: '2014-11-25T17:39:50+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=49'
permalink: /2014/11/25/quickly-moving-fsmo-around-domain-controllers-with-powershell-ad-cmdlets/
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
    - 'Active Directory'
    - FSMO
    - Move-ADDirectoryServerOperationMasterRole
---

Of course, Microsoft provides us with a cmdlet to move FSMO roles between Domain Controllers… But as often, this doesn’t seem to be designed by people who use it.

First, the name of the cmdlet is lengthy:[ Move-ADDirectoryServerOperationMasterRole](http://technet.microsoft.com/en-us/library/ee617229.aspx)  
Second, you must indicate every FSMO role you want to move amongst the following values: PDCEmulator, RIDMaster, InfrastructureMaster, SchemaMaster or DomainNamingMaster.  
Of course the wording is slightly different (again) between this cmdlet and the ntdsutil you may have used in Windows 2003 or in Windows 2008 before powershelling everything,

Fortunately the documentation tells us that we can use numbers from 0 to 4 instead of the name. But there is a little nice trick to notice instead of trying to remember which role maps to which number. The FSMO roles for a domain are at the beginning of the sequence, whereas the forest FSMO roles are 3 and 4.  
Therefore in a forest where you have top and child domains, you may just to use:

```
<pre class="lang:ps decode:true " title="Move Domain FSMO Only">Move-ADDirectoryServerOperationMasterRole MyTargetDC (0..2)
```

for domain-limited move  
or

```
<pre class="lang:ps decode:true " title="Move ForestFSMO">Move-ADDirectoryServerOperationMasterRole MyTargetDC (0..4)
```

for forest-wide move.

You may also notice the parenthesis set to create an array from a range