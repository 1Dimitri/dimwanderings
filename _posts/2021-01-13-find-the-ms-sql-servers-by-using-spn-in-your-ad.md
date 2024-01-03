---
id: 1061
title: 'Find the MS SQL Servers by using SPN in your AD'
date: '2021-01-13T12:12:53+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=1061'
permalink: /2021/01/13/find-the-ms-sql-servers-by-using-spn-in-your-ad/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1063;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2021/01/Get-ADObject-Powershell-Picture.png
categories:
    - 'Active Directory'
    - batch
    - 'Powershell Engine'
    - troubleshooting
tags:
    - Get-ADObject
    - ldap
---

In an [old post](https://dimitri.janczak.net/2015/08/21/find-the-rds-licensing-servers-in-your-domain/), I was writing about the commands you could use to retrieve the list of your RDS Licensing servers in a domain.

Generalizing this LDAP Filter example, you can retrieve a list of Service Principal Names enabled accounts. The examples are as follows:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">dsquery * -filter "(servicePrincipalName=*)"
```

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">Get-ADObject -Filter { serviceprincipalname -like '*' }
# or directly translated as:
# Get-ADObject -LDAPFilter "(servicePrincipalName=*)"
```

In the powershell syntax with Filter you can not write something like `{ servicePrincipalName -ne $null}`.

If you want to narrow down your searches to the SQL Servers for which Kerberos is enabled you can use

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">dsquery * -filter "(servicePrincipalName=MSSQLSvc/*)"
```

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">Get-ADObject -Filter { serviceprincipalname -like 'MSSQLSvc/*' }
# or directly translated as:
# Get-ADObject -LDAPFilter "(servicePrincipalName=MSSQLSvc/*)"
```

Refinements to that query could be to target SQL Servers SPN on computer objects when you are using SYSTEM / NT Service accounts or on the contrary user accounts when you are using domain users. Such filters would be:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">dsquery -filter "(&(objectcategory=computer)(servicePrincipalName=MSSQLSvc/*))"
REM or dsquery -filter "(&(objectcategory=user)(servicePrincipalName=MSSQLSvc/*))"

```

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">Get-ADObject -Filter { (serviceprincipalname -like 'MSSQLSvc/*') -and  (objectcategory -eq 'computer') }
# or directly translated as:
# Get-ADObject -LDAPFilter "(&(objectcategory=computer)(servicePrincipalName=MSSQLSvc/*))"
```

Additional complex filters can be found in the [Active Directory filter Powershell help page](https://docs.microsoft.com/en-us/previous-versions/windows/server/hh531527(v=ws.10)).

Get-ADObject is a more generic Powershell cmdlet as its Get-ADComputer and Get-ADUser counterparts, as it doesn’t assume an objectCategory hence its broader use but more complex syntax.

Always remember that the syntax of the LDAPFilter immediately matches the one you find in the dsquery command, whereas the Filter option must be written using Powershell Script block Syntax.