---
id: 517
title: 'Ambiguous Fuzzy search in AD'
date: '2016-11-08T17:46:47+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=517'
permalink: /2016/11/08/ambiguous-fuzzy-search-in-ad/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"527";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/11/AD-New-User-Creation.jpg
categories:
    - 'Active Directory'
tags:
    - 'ambiguous name resolution'
    - cmdlet
    - domain
    - identity
    - 'LDAP search'
    - powershell
    - snippet
    - user
---

Working in big AD forests and domains can be sometimes challenging, as powershell cmdlets require you perfect identity matches. In an AD of a global corporation, where you have tons of John, Sam, Doe, Smith, Martin, Schmidt , finding that information may be time consuming: is the name given a first name, a last name, a maiden’s name… You have to do Ambiguous Fuzzy search in AD.

Instead of being embarassed asking Here’s a little snippet that takes as input

- a user name
- a switch

If the switch ‘strict’ is not present,

- An AD search will be created to find the best match against first names, last names, SAM Account names. In fact it does [ambiguous name resolution](http://social.technet.microsoft.com/wiki/contents/articles/22653.active-directory-ambiguous-name-resolution.aspx).
- If only one account is found, the operation is processed
- If multiple account are found, an error is thrown showing which account are matching your criteria, so you can be more precise when relaunching your cmdlet

If the switch ‘strixt’ is present, the user name must be a unique perfect matching identity as the parameter will be used as a parameter to Get-ADUser

```
# AD search ambiguous

Function Do-ADUserSomething {
param(
[Parameter(Mandatory=$true)]
[string]$User,
[switch] $strict
)
 if ($strict) {
   $u = Get-ADUser -Identity $user
} else {
 $u = Get-ADUser -LDAPFilter "(anr=$user)"
}
 if ($u) {
   if ($u -is [array]) 
   {
     throw "Found multiple users matching (anr=$user): $($u.samAccountName -join ', ' )"
   }
   else
   {
   #TO DO: Add your code using $u 
   }
 }
 else 
 {
  throw "No user found matching criteria (anr=$user)"
 }

}
```

You just need to fill in the #TO DO line with your adhoc code.

You may want to use return values or another mechanism like return values or warning instead of throwing up an exception in case there is more than one user which matches your criteria. You can also replace GEt-ADUser with a more generic Get-ADObject cmdlet if your work include contacts or groups.