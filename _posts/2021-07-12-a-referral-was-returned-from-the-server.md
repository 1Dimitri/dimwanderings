---
id: 1088
title: 'Get-ADDomainController, Get-ADUser, Get-ADComputer, Get-ADObject: A referral was returned from the server'
date: '2021-07-12T13:41:02+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=1088'
permalink: /2021/07/12/a-referral-was-returned-from-the-server/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1089;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2021/07/Get-ADDomainController-Exception-A-referral-was-returned-from-the-server.jpg
categories:
    - 'Active Directory'
tags:
    - powershell
    - troubleshooting
---

When you are using Active Directory Powershell cmdlets such as Get-ADDomainController, Get-ADUser, Get-ADComputer, Get-ADObject, you may receive the following error message:

```
<pre class="wp-block-code">```
A referral was returned from the server
```
```

Such an exception is linked to the difficulty those commands have to work in a multi-domain environment, whether that multi-domain be a single forest with multiple child domains, various trust relationships.

Let’s imagine you have a forest with domains called Top, Child1 and Child2. If your user is Top\\myadmin and your trying to get information for the domain controller dc1child1 in the child domain child1, you may want to issue the following command from a machine in the “Top” domain

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">$MyDCInfo = Get-ADDomainController dc1child1
```

The answer will be:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">TerminatingError(Get-ADDomainController): "A referral was returned from the server"
Get-ADDomainController : A referral was returned from the server
At line:1 char:94
+ ... oryServers | % { $domaincontrollers[$_]=Get-ADDomainController $_  }}
+                                             ~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (dc1child1.child1.com:ADDomainController) [Get-ADDomainController], ADException
    + FullyQualifiedErrorId : 
ActiveDirectoryServer:8235,Microsoft.ActiveDirectory.Management.Commands.GetADDomainController
Get-ADDomainController : A referral was returned from the server
```

To circumvent this, you must specify a target server from the domain you want the object from. In our case, you will have to type,:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">$MyDCInfo = Get-ADDomainController dc1child1 -Server dc1child1
```

Here the name is repeated twice, as the object we are searching for is also the name of the server you’re asking to answer. But in such a case as a domain user, this would be:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">$myuser = Get-ADUser mychilduser -Server dc1child1
```

Note that to find a correct Domain Controller for a given domain, Get-ADDomainController with the discover switch may be very helpful.