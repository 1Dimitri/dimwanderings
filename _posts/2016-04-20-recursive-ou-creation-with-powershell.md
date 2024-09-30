---
id: 431
title: 'Recursive OU creation with Powershell'
date: '2016-04-20T10:31:41+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=431'
permalink: /2016/04/20/recursive-ou-creation-with-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"18";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/08/powershell-logo-e1410861403696.jpg
categories:
    - 'Active Directory'
tags:
    - powershell
    - PSDrive
---

In basic AD Powershell, when creating Organizational Units (OU), you are often referred to the [New-ADOrganizationalUnit](https://technet.microsoft.com/en-us/library/ee617237.aspx) cmdlet, which is fine if you must create one OU or multiple unrelated OUs, but recursive OU creation with Powershell can be verbose.

If you must create multiple OUs which are child, child of child, the cmdlet becomes unpractical: you must concatenate strings to create the DN path or copy/paste them.

A little unknown gem is the powershell ‘AD:’ drive, which allows to traverse the Active Directory structure like a file system. Creating OUs just then become a matter of cd and mkdir commands.

The only gotcha you must be aware is that the name of the object you create must start with the object type. For an OU, it is ‘OU=’. For a container it would be ‘CN=’. In fact, you are using RDN (Relative Distinguished Names) in each folder.

Therefore, at the top of the drive, to enter the domain, you would type in ‘cd .\\dc=mydomain,dc=local’ if your domain is mydomain.local

Here is an example:

```
PS C:\> cd AD:\
PS AD:\> ls

Name                 ObjectClass          DistinguishedName
----                 -----------          -----------------
Configuration        configuration        CN=Configuration,DC=foobar,DC=net
Schema               dMD                  CN=Schema,CN=Configuration,DC=foobar,DC=net
foobar               domainDNS            DC=foobar,DC=net
DomainDnsZones       domainDNS            DC=DomainDnsZones,DC=foobar,DC=net
ForestDnsZones       domainDNS            DC=ForestDnsZones,DC=foobar,DC=net


PS AD:\> cd '.\DC=foobar,DC=net'
PS AD:\DC=foobar,DC=net> mkdir 'OU=mytopOU'
```

Adding nested Organizational Units can be then done as:

```
PS AD:\DC=foobar,DC=net> cd '.\OU=mytopOU,DC=foobar,DC=net'
PS AD:\OU=mytopOU,DC=foobar,DC=net> mkdir '.\OU=mysubOU'
...
```

You can of course use Import-CSV and related cmdlets to import the list of objects that you want to create.

Alternatively you may also use XML format to import an ordered nested structure.