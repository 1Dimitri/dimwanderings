---
id: 458
title: 'List Domain Controllers Forest-Wide'
date: '2016-07-17T10:14:03+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=458'
permalink: /2016/07/17/list-domain-controllers-forest-wide/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"459";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/07/AD-NTDS-Settings.jpg
categories:
    - 'Active Directory'
tags:
    - 'Domain Controllers'
    - powershell
    - snippet
---

In a [previous post, ](http://dimitri.janczak.net/2014/11/21/getting-all-domain-controllers-in-a-forest/)I’ve put a snippet using powershell to be able to list domain controllers forest-wide. Here’s a variation if you have some legacy Powershell 2.0, etc. The algorithm takes advantage of the ADSI inteface and the AD Searcher object instead of relying on some built-in powershell cmdlets.

We first get the RootDSE object, from which we extract the name of the forest.

```
<pre class="lang:ps decode:true" title="List Domain Controllers Forest-wide">try {
$RootDSE=([ADSI]"LDAP://RootDSE")
$ForestRootDomain=$RootDSE.rootDomainNamingContext        
$ldapQuery = "(&(objectClass=nTDSDSA))"
              $ObjAD = new-object System.DirectoryServices.DirectoryEntry
              $ADSearcher = new-object system.directoryservices.directorysearcher –argumentlist $ObjAD,$ldapQuery
              $Root = New-Object DirectoryServices.DirectoryEntry "LDAP://CN=Sites,CN=Configuration,$ForestRootDomain"
              $ADSearcher.SearchRoot = $Root
              try 
              {
                    $QueryResult = $ADSearcher.findall()
                    $QueryResult | 
                    foreach {
                          $ldapDN=$_.Path.replace("LDAP://CN=NTDS Settings,","")
                          $ldapDN
        
                    }
              }
              catch 
              {
                  write-host "Exception raised by SearchDirectory:"
                  write-host $_ -fore red
                  break
              }
}
   catch 
              {
                  write-host "Exception raised by RootDSE retrieval:"
                  write-host $_ -fore red
                  break
              }
```

From there we retrieve all the NTDS Settings objects from the full forest and we strip the NTDS Settings prefix to get only the distinguished name of the computer object of the Domain Controller.