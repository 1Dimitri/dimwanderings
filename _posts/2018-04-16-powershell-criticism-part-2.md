---
id: 864
title: 'Powershell criticism: Part 2'
date: '2018-04-16T13:29:24+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=864'
permalink: /2018/04/16/powershell-criticism-part-2/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:869;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/04/ForEach-Object-Loop-Powershell.png
categories:
    - 'Powershell Engine'
tags:
    - $_
    - cleartext
    - cmdlet
    - language
    - PSItem
    - ranting
    - security
---

In a previous post, we looked at some [powershell oddities when it comes to Credential use](https://dimitri.janczak.net/2018/04/09/powershell-criticism-part-1/). In this “Powershell criticism: part 2”, let’s look at some language construction irritating way to express things shortly, when doing nested enumeration.

As a language used by system administrators, you’ll use Powershell in two forms:

- Scripts, functions or modules you willl resume where the code spans a few lines or more, and aliases are not welcomed. Brevity is of no concern in this scenario;
- One-liners where the powershell statement has to be as short as possible and where aliases are more than welcome.

In the second scenario, nesting loops over collections can be less than efficient. Everybody has wondered what $\_ meant when the first ForEach-Object construction has been encountered. The traditional example would be

```
<pre class="lang:ps decode:true" title="ForEach-Object">Get-Process | % { $_.Name }
```

As mentioned in the [ForEach-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/foreach-object?view=powershell-6) page, the $\_ was often verbose when extracting a property, therefore this ScriptBlock construct was given an Operation alternate form in Powershell 3.0. At the same time, the name $PSItem was introduced as $\_ seemed a bit magical. A good summary of all these[ Powershell 3.0 Foreach-Object enhancements](https://blogs.msdn.microsoft.com/mvpawardprogram/2013/04/15/working-with-the-new-psitem-automatic-variable-in-windows-powershell-3-0/) are summarized in that post.

However, when you are writing onelines, you often use multiple piped cmdlets and when you’re using multiple ForEach-Object constructions to enumerate a collection within a collection, things aren’t so easy to write in a concise manner. $\_ and $PSITem refer to the most inner loop and not to any outer collection.

Let’s take the services and try to build easily the list of services depended on. IN a construction like below, the $\_ cannot refer to the initial service object unless you assign it in a named variable:

```
<pre class="lang:ps decode:true " title="Enumerator within enumator variable access problem">Get-SErvice | % { $_.ServicesDependedOn | % { $_.Name }  }
```

You must add an intermediate variable

```
<pre class="lang:ps decode:true" title="Nested collection enumeration">Get-Service | % { $Svc = $_ ; $_.ServicesDependedOn | % { "For Service = $($svc.Name) -> $($_.Name)" } }
#or
foreach ($svc in Get-Service) {  $svc.ServicesDependedOn | % { "For Service = $($svc.Name) -> $($_.Name)" } }
```

which is no issue in .ps1 file but defeats the purpose of brievity for onelines

An enhancement proposal to access variables when nested enumeration is needed could be:

1. Create automatic variable named $\_\_, $\_\_ four outer scopes
2. Create a $PSItems array where $PSItems\[0\] would be the current scoped variable, aka $\_ or $PSItem, $PSItem\[1\] the outer level, $PSItems\[2\] the outer outer level…

In the next post, we’ll go back to a built-in cmdlet which doesn’t work the way everybody would like it to work.