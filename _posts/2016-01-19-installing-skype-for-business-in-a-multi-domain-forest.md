---
id: 398
title: 'Skype for Business installation in a multi-domain forest'
date: '2016-01-19T20:05:28+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=398'
permalink: /2016/01/19/installing-skype-for-business-in-a-multi-domain-forest/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"399";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/01/Skype_for_Business.png
categories:
    - 'Lync Skype for Business'
tags:
    - 'Active Directory'
    - 'Global Catalog'
    - schema
    - troubleshooting
---

## Issues

Although the Lync 2013 / Skype for Business 2015 installation seems simple, there are a few caveats in some scenarios

If your forest is made of several domains:

- You cannot run the initial schema update from a machine which is in any child domain. The following steps must be run in the root domain : 
    - [Install-CsAdServerSchema](https://technet.microsoft.com/en-us/library/gg398681.aspx)
    - [Enable-CsAdForest](https://technet.microsoft.com/en-us/library/gg425713.aspx) but you can target a child domain.
- The only action you must perform in the child domain is: 
    - [Enable-CsAdDomain](https://technet.microsoft.com/en-us/library/gg412764.aspx)
- Imagine you have an Active Directory is made of a top empty domain and a child where every real user resides.  
    When you prepare the universal groups and want to have them in the child domain, you cannot use the setup graphical interface. The setup wizard makes you believe you can specify a domain which is not the root domain. ```
    <pre class="">But in all cases I saw, this fails with an error message saying "Error: Object reference not set to an instance of an object. Type: NullReferenceException at Microsoft.Rtc.Management.Deployment.LcForest.PrepareForest()".
    ```
    
    You can however bypass this problem by using the Lync shell.  
    Use the Enable-CsAdForest with the GroupDomain and GroupDomainController switches to manually specify the domain and the target Global Catalog to use.  
    In child.forest.dom this would mean:
    
    ```
    <pre class="lang:ps decode:true " title="Enable-CsAdForest example">Enable-CsAdForest -GroupDomain child.forest.dom -GroupDomainController DC-in-child.child.forest-domain -verbose -report myreport.html
    ```

## Complete Skype for Business installation sequence

A complete Active Directory preparation sequence would then be:

```
<pre class="lang:ps decode:true" title="Skype For Business installation sequence"># (member server of domain.dom)
Install-CsAdServerSchema  -verbose -report step1.html
Enable-CsAdForest -GroupDomain child.domain.dom -GroupDomainController child-dc.child.domain.dom -verbose -report step2.html

#  (member server of child.domain.dom)
Enable-AdCsDomain -verbose -report step3.html
```

You can of course name your report html pages differently. You can include date, time or machine name. You canalso test the final result with [Get-CsAdDomain](https://technet.microsoft.com/en-us/library/gg398453.aspx).

It is also worthwhile to note that you may end up with problems if your AD groups start with ‘CS’ as these letters are used for the Skype for Business built-in groups.