---
id: 575
title: 'Sharepoint group copy users'
date: '2017-01-17T13:51:36+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=575'
permalink: /2017/01/17/sharepoint-group-copy-users/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"577";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/01/sharepoint-users.jpg
categories:
    - Sharepoint
tags:
    - permissions
    - powershell
    - snippet
---

In Sharepoint sometimes users have difficulties to grasp the permissions system.

You may have to change users membership quite a few times because ‘user X must have the same rights as User Y’.

If you have more thtan a couple of users, it can become quickly very boring.

The following snippet from the ‘lazy admin’ collection just does that: take a sharepoint group copy users to another group.

```
<pre class="lang:ps decode:true " title="Sharepoint group copy users to another group"># Add-PSSnapin Microsoft.SharePoint.PowerShell 
<#
.SYNOPSIS 
Copy sharepoint users between groups
.DESCRIPTION 
Copy users in a Sharepoint group to another Sharepoint Group
.INPUTS 
  WebURL: site URL
  SourceGroupName: name of the source group
  TargetGroupName: name of the target group
.OUTPUTS 
 None
.EXAMPLE 
Copy-SPGroupToGroup -WebURL "https://foobar/" -SourceGroupName "VIPs" -TargetGroupName "Project Managers"
.NOTES 
 use Sitegroups in case group has not been granted permissions on the site
#>

Function Copy-SPGroupToGroup {
param(
[Parameter(Mandatory=$true,Position=2)]
[string]$WebURL,
[Parameter(Mandatory=$true,Position=0)]
[string]$SourceGroupName,
[Parameter(Mandatory=$true,Position=1)]
[string]$TargetGroupName
)

$web = Get-SPWeb $WebURL

if ($web -eq $null) {
  throw "$WebURL does not match a site or site collection"
}

#Get the Source and Target Groups

$SourceGroup = $web.Sitegroups | where {$_.name -eq $SourceGroupName }
$TargetGroup = $web.Sitegroups | where {$_.name -eq $TargetGroupName }

if ($SourceGroup -eq $null) {
    throw "SourceGroup $SourceGroupName not found"
} 

if ($TargetGroup -eq $null) {
    throw "Target Group $TargetGroupName not found"
} 

#Iterate through each users in the source group

    foreach ($user in $SourceGroup.users)

    {
       $TargetGroup.AddUser($user)
        Write-Verbose "Copied $user from $SourceGroup to $TargetGroup"
        #To move users, Just remove
        #$SourceGroup.RemoveUser($user)
    }

}

```