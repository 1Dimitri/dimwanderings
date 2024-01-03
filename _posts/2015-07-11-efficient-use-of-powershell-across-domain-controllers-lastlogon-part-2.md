---
id: 238
title: 'Efficient Use of Powershell across Domain Controllers: LastLogon Part 2'
date: '2015-07-11T14:44:17+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=238'
permalink: /2015/07/11/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-2/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:234;s:11:"_thumb_type";s:10:"attachment";}'
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

Our [previous script](http://dimitri.janczak.net/2015/07/09/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-1/) was simple but not quick. Our second attempt will try to reduce the time it takes to gather all the pieces.

First of all we may imagine that our use of the hashtable in Powershell may be inefficient as we start from 0 object and are continuously adding new users, whereas we approximately know the number of users from any Get-ADUser count returns. The approximation could only be wrong if a bunch of new users were created on a single Domain Controller and not yet replicated (In this case your RID Master will have its share of workload).

Some research on the Internet may help you to find [this kind of thread](http://stackoverflow.com/questions/7523143/powershell-2-and-net-optimize-for-extremely-large-hash-tables) where you’ll learn that you can initialize a hashtable with a predefined number of elements, using the following syntax:

```
<pre class="lang:ps decode:true " title="Powershell Hashtable alternate constructor">New-Object Hashtable 100000
```

Then we may rewrite our loop to gather all the information and then do computation:

```
<pre class="lang:ps decode:true" title="Powershell Get-ADUser across Domain Controllers, second attempt">$DCs = Get-ADDomainController -Filter '*'
$r = New-Object Hashtable ($DCs.Count)

Write-Host 'Getting All Users LastLogon for each DC'
#Change the starting DN below:
Measure-Command {
foreach ($dc in $dcs) { $r[$dc] = Get-ADUser -Server $dc.HostName -Properties LastLogon -SearchBase 'OU=Users,dc=mydomain,dc=local' -SearchScope OneLevel -Filter *} 
}


$k1 = $r.Keys | Select -First 1
$nb = ($r[$k1]).Count
$t = New-Object Hashtable $nb

Write-Host "Computing across $nb users..."

Measure-Command {
foreach ($dc in $DCs) { 
	foreach ($u in $r[$dc]) {
		if ($t.ContainsKey($u.name)) {
			if ($t[$u.name] -lt $u.LastLogon) {
				$t[$u.name] = $u.LastLogon
			} 
		} 
		else { 
			$t[$u.name]=$u.LastLogon
		}
	} 
}

}
```

A run of this program will output this kind of figures (8000 users on a dozen of domain controllers located across the world):[![GetADUser-MeasureCommand-Attempt2](http://dimitri.janczak.net/wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt2.png)](http://dimitri.janczak.net/wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt2.png)

It is better than our first attempt. However if we look at the resources used during the run, we may still be surprised that there is connection to the network during the entire time the script is run. If you are not convinced, launch the resource monitor and filter the ‘powereshell’ process. With our script, you’ll see that each Domain Controller is still contacted during the ‘Computing’ part of our script.

Therefore our information retrieval is not as optimized as we though. That will be a quest for [part 3](http://dimitri.janczak.net/2015/07/14/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-3/).