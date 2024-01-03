---
id: 52
title: 'Getting all domain controllers in a forest'
date: '2014-11-21T10:53:53+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=52'
permalink: /2014/11/21/getting-all-domain-controllers-in-a-forest/
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
    - domain
    - forest
    - get-addomain
    - get-adforest
---

Using something like Get-ADDomainController -Forest would be something too easy for module designers at Microsoft. Or does Microsoft no longer have multi-domain forests in its internal IT?

Never mind, here is a nice snippet to retrieve that you may want to adapt to the fields you need to retrieve

```
<pre class="lang:default decode:true " title="Retrieve all domain controllers from a forest">(Get-ADForest).Domains | % { Get-ADDomainController -Discover -DomainName  $_ } | % { Get-ADDomainController -server $_.Name -filter * } | Select Name, Domain, Forest, IPv4Address, Site | ft
```