---
id: 240
title: 'Efficient Use of Powershell across Domain Controllers: LastLogon Part 3'
date: '2015-07-14T20:45:07+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=240'
permalink: /2015/07/14/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-3/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:269;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - 'optimizing powershell series'
    - 'Powershell Engine'
tags:
    - 'Active Directory'
    - 'Domain Controllers'
    - Hashtable
    - LastLogon
    - LastLogonTimeStamp
    - optimization
    - remote
    - tutorial
---

After our simplistic approach in the [1st part](http://dimitri.janczak.net/2015/07/09/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-1/) and our discoveries about optimization in [part 2](http://dimitri.janczak.net/2015/07/11/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-2/), it is time to gather our learnings to lower the total time our script will run:

- We know that hashtables can be initialized with the number of elements they will approximately contain to avoid costly memory operations
- We know that we should avoid keeping references to Get-ADUser as the code may try to reconnect to the distant Domain Controllers
- Let’s remind ourselves that we only need to get the numeric LastLogon attribute and observe that there is a Domain Controller which by default is less costly to interrogate, that’s the ‘nearest Domain Controller’ according to the DC Locator algorithm.

Therefore we start to think about a optimized way of doing things:

1. Get the users and their total figure from the nearest Domain Controller
2. Build a pre-allocated hashtable with that piece of information. Since we have already one LastLogon attribute, let’s fill in that hashtable fully
3. Then iterate through all other domain controllers other than the one we already targeted to update the LastLogon attribute accordingly
4. Finally, convert the LastLogon to a Date/Time format instead of that geeky number of ticks.

Here is the script:

```
<pre class="lang:ps decode:true ">$DCs = (Get-ADDomainController -Filter '*').HostName
$NearestDC= (Get-ADDomainController -Discover -NextClosestSite).HostName | Select -First 1
Write-Host "Getting users from nearest DC $nearestDC"
Measure-Command {
$ADUsers = Get-ADUser -Server $NearestDC -Properties LastLogon -SearchBase 'OU=Users,dc=fabrikam,dc=com' -SearchScope OneLevel -Filter *
}

Write-Host 'Populating reference data'
Measure-Command {
$users = New-Object HashTable ($ADUsers.Count)
$ADUsers | % { $users.Add($_.Name,$_.LastLogon) }
}

Write-Host 'Getting All Users LastLogon for each DC'
Measure-Command {
foreach ($dc in $dcs) { 
if ($dc -ne $nearestDC) { 
Write-Host "Getting All Users LastLogon from $DC"
	Get-ADUser -Server $dc -Properties LastLogon -SearchBase 'OU=Users,dc=fabrikam,dc=com' -SearchScope OneLevel -Filter * `
	   | % { if ($_.LastLogon -gt $users[$_.name]) { $users[$_.name] = $_.LastLogon} }
	} 
  }
}

Write-Host 'Converting Date/Time'

Measure-Command { 
foreach ($key in @($users.keys)) { If ($users[$key]) { $users[$key] = [datetime]::FromFileTime([int64]::Parse($users[$key])) } }
}
```

Note that little tweak to update an hashtable with the array of keys created at the last line, since you cannot update directly such an object if you’re using the regular iterator.

Finally the results are quite interesting. From initial tests running during 9 hours we are at less than 5 minutes for this dozen Domain Controllers, 8000 users thing.

[![GetADUser-MeasureCommand-Attempt3](http://dimitri.janczak.net/wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt3.png)](http://dimitri.janczak.net/wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt3.png)

Some of you may wonder why not use workflow and parallel processing: the main gotcha is that at the end you must update the LastLogon value, which may be erased depending on which parallel task finishes first. If you fully want to go parallel, you have to think differently and use immutable data like in functional programming.