---
id: 1056
title: 'WMI filters to target sites and non Domain Controllers'
date: '2020-12-23T15:25:58+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=1056'
permalink: /2020/12/23/wmi-filters-to-target-sites-and-non-domain-controllers/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1057;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2020/12/WMI-Filter-Non-DC.png
categories:
    - 'Active Directory'
    - 'Windows OS'
tags:
    - filter
    - GPO
    - 'Group Policy Object'
    - wmi
---

Designing Group Policy Objects is sometimes tricky. Let’s review some useful WMI filters in Group Policy Objects for sites and domain controllers inclusion or exclusion.

When linking Group Policy Objects, you are always bound to the LSDOU rule, which means that GPOs are processed in the following order:

- L for Local GPOs
- S for Active Directory Sites
- D for Domain
- OU for… OU, Organizational Units

Basically, a GPO that targets an OU will always takeover precedence over a GPO assigned to the domain, which itself takes precedence over a Site.

By cleverly ordering GPOs, you usually have no problem to get the result you want when 2 GPOs target the same setting.

However there are edge cases, such as Domain Controllers being in their own OU. Microsoft clearly tells you that [moving Domain Controllers to other OUs or using sub-OUs to divide your domain controllers is an unsupported scenario](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc728418(v=ws.10)).

What can you do then when you want to have different GPOs for Domain Controllers and the “rest of computers”?

The Computers built-in folder is a not an OU but a simple container and a container does not support GPO link. You may then think to link to the domain, and override the policy at the Domain Controllers OU, but it is not always feasible if you want the setting not to apply, instead of being applied with another value.

You may then think to move all your computers under a “TopRootOU” but this is not always feasible if the structure is already in place, or for those reason you cannot afford that root OU:

- you do not want to have computers getting no GPO while they are sitting in the Computers container
- you want to keep the GPO link to the domain for better readability
- or avoid restructuring your multiple root OUs already in place

In that case, designing a WMI filter that target computers which are not Domain Controllers allow to achieve this:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">SELECT * FROM Win32_ComputerSystem Where DomainRole <> 4 AND DomainRole <> 5
```

<figure class="wp-block-image size-large">[![WMI filter To select non domain controllers](https://dimitri.janczak.net/wp-content/uploads/2020/12/WMI-Filter-Non-DC.png)](https://dimitri.janczak.net/wp-content/uploads/2020/12/WMI-Filter-Non-DC.png)</figure>In a related area, the GPO processing order makes it difficult to have different GPOs for different domain controllers based on dynamic criterias since sub-OUs within the “Domain Controllers” OU are not supported, and Site GPOs are overriden by OU-linked GPOs.

You could rely on Group Membership or Computer Account in the Security Tab but this requires an additional step when promoting or demoting a domain controller. Recreating sites with a WMI Filter can however be accomplished.

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="sql" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">Select * FROM Win32_NTDomain WHERE (DcSiteName = 'YourSite' AND DomainName = 'DomainShortName')
```

The DomainName part is needed when computer or users accounts from multiple domains are living next to each other on the same subnets and sites. Note that if your sites follow some convention, you can use the like operator, e.g.

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="sql" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">Select * FROM Win32_NTDomain WHERE (DcSiteName like 'Start%' AND DomainName = 'DomainShortName')
```

Domain Short name is the name of your domain without any dot, aka the NetBios name, not the fully qualified domain name.