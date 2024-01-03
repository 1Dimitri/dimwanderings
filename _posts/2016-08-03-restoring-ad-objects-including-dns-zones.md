---
id: 470
title: 'Restoring AD objects including DNS zones'
date: '2016-08-03T15:07:47+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=470'
permalink: /2016/08/03/restoring-ad-objects-including-dns-zones/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"473";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/08/Recycle-Bin.jpg
categories:
    - 'Active Directory'
tags:
    - 'granular restore'
    - powershell
    - restore
    - snippet
---

Since the initial times of [Windows 2003](https://msdn.microsoft.com/en-us/library/ms677923(v=vs.85).aspx), things have changed. Restoring AD objects, including DNS Zones become simpler with each release of Windows Server. Therefore let’s see how you can restore users, computers, organizational units and DNS zones nowadays.

## Restoring AD objects for standard classes

Since Windows 2008 R2,[ Get-ADObject](https://technet.microsoft.com/en-us/library/ee617198.aspx) has a nice switch called IncludeDeletedObjects which may be of help.

```
<pre class="lang:ps decode:true " title="Restore AD object">Import-module activedirectory # If you are under Windows 2008(R2)
# Get the object(s) for the zone by using
$obj = Get-ADObject -filter 'samaccountname -eq "myuser" ' -includedeletedobjects
# $obj = Get-ADObject -filter 'name -like "*myuser*" ' -includedeletedobjects
# Display matching object(s) for checking purposes
$obj
# Restore it
$zone | Restore-ADObject

# Restore the records
# At this stage, the zone is still named  ..Deleted-zone
Get-ADObject -filter 'isdeleted -eq $true -and lastKnownParent -like "DC=..Deleted-*,CN=MicrosoftDNS,DC=ForestDnsZones,DC=mydomain,DC=local"' -includedeletedobjects -searchbase "DC=ForestDnsZones,DC=contoso,DC=com" | restore-adobject
Get-ADObject -filter 'isdeleted -eq $true -and lastKnownParent -like "DC=..Deleted-*,CN=MicrosoftDNS,DC=ForestDnsZones,DC=mydomain,DC=local"' -includedeletedobjects -searchbase "DC=DomainDnsZones,DC=contoso,DC=com" | restore-adobject

# Rename the zone 'MyZone'
Rename-ADObject "DC=..Deleted-myzone,CN=MicrosoftDNS,DC=ForestDnsZones,DC=mydomain,DC=local" -newname "myzone"
# or
Rename-ADObject "DC=..Deleted-myzone,CN=MicrosoftDNS,DC=DomainDnsZones,DC=mydomain,DC=local" -newname "myzone"

#end
```

Also, In Windows 2012(r2), the [Administrative Center ](https://technet.microsoft.com/en-us/windows-server-docs/identity/ad-ds/get-started/adac/advanced-ad-ds-management-using-active-directory-administrative-center--level-200-)has also an interface to do so.

## Restoring AD objects for DNS zones and records

To restore DNS records, there is a difference you should know. Before being moved to the Active Directory deleted objects containers, the zone is first renamed with a ‘..Deleted’ prefix. And there is no GUI as of today.

Therefore to restore such a zone, you must:

1. Find the ‘..Deleted’ zone object
2. Restore that object
3. Restore every deleted record object in that zone
4. Rename the zone to its previous name

Examples of command are then:

```
<pre class="lang:ps decode:true " title="Restore AD integrated DNS zones">Import-module activedirectory # If you are under Windows 2008(R2)
# Get the object(s) for the zone by using one of the following 
$zone = Get-ADObject -filter 'isdeleted -eq $true -and name -like "*..Deleted-*"' -includedeletedobjects -searchbase "DC=ForestDnsZones,DC=mydomain,DC=local" -property * #if Forest wide
$zone = Get-ADObject -filter 'isdeleted -eq $true -and name -like "*..Deleted-*"' -includedeletedobjects -searchbase "DC=DomainDnsZones,DC=mydomain,DC=local" -property * #if domain wide
# Display matching object(s)
$zone
# Restore it
$zone | Restore-ADObject

# Restore the records
# At this stage, the zone is still named  ..Deleted-zone
Get-ADObject -filter 'isdeleted -eq $true -and lastKnownParent -like "DC=..Deleted-*,CN=MicrosoftDNS,DC=ForestDnsZones,DC=mydomain,DC=local"' -includedeletedobjects -searchbase "DC=ForestDnsZones,DC=contoso,DC=com" | restore-adobject
Get-ADObject -filter 'isdeleted -eq $true -and lastKnownParent -like "DC=..Deleted-*,CN=MicrosoftDNS,DC=ForestDnsZones,DC=mydomain,DC=local"' -includedeletedobjects -searchbase "DC=DomainDnsZones,DC=contoso,DC=com" | restore-adobject

# Rename the zone 'MyZone'
Rename-ADObject "DC=..Deleted-myzone,CN=MicrosoftDNS,DC=ForestDnsZones,DC=mydomain,DC=local" -newname "myzone"
# or
Rename-ADObject "DC=..Deleted-myzone,CN=MicrosoftDNS,DC=DomainDnsZones,DC=mydomain,DC=local" -newname "myzone"

#end
```

It is interesting to note that as long as the DNS zoned is called ‘..Deleted-XXX’ it doesn’t appear in any DNS management tools.