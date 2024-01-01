---
id: 236
title: 'Efficient Use of Powershell across Domain Controllers: LastLogon Part 1'
date: '2015-07-09T21:11:17+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=236'
permalink: /2015/07/09/efficient-use-of-powershell-across-domain-controllers-lastlogon-part-1/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"234";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt2.png
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

One of the frequent reports you’re asked to do when you’re administering an Active DIrectory is to know when a user has connected for the last time or how long has a computer account been inactive.

In the infancy of the Active Directory, Microsoft faced these requirements too. In Windows 2000 times, they didn’t fully anticipate how often the need will be raised by their customers but were more worried about limiting the amount of data each Domain Controller would replicate since leased lines were not cheap, CPU power was limited… So they come with that simple solution where each Domain Controller would record the time of lastlogon in an attribute called Lastlogon (make sense) but this attribute would not be replicated. To get your report, you would have to collect the attribute for every Domain Controller and do the computation by yourself.

When Windows 2003 came out, the balance shifted a bit towards the easiness of getting a report, but this attribute was seen more as a tool to get inactive accounts so the real-time aspect was not top priority. That’s why they introduced the lastlogontimestamp attribute which was then a replicated attribute, but which wasn’t updated every time still to avoid too much overhead. This limitation is explained [here](http://blogs.technet.com/b/askds/archive/2009/04/15/the-lastlogontimestamp-attribute-what-it-was-designed-for-and-how-it-works.aspx), with a way to make this attribute updated sooner.

As mentioned in that article, if you need more accurate data, one way is to parse security log, the other one is to revert back to good old Windows 2000 times by getting the LastLogon attribute from every Domain Controller. However, as time passes by, new methods to do so have emerged, and we’ll try in this series of articles to do real administrative powershell by trying to collect that information quickly even for non-test Active Directories: let’s take a 8000-users AD spanned over the world with a dozen of Domain Controllers put in different datacenters and regional offices.

As you are as lazy as I am, let’s start by doing a simple loop to collect the information one user at a time on each DC, like any newbie in Powershell would easily do. If you need a solution, you can find such an implementation [at this page.](https://www.interworks.com/blog/trhymer/2014/01/22/powershell-get-last-logon-all-users-across-all-domain-controllers)

```
<pre class="lang:ps decode:true  " title="Powershell simple LastLogon collection attempt">function Get-LastLogon()
{
  $dcs = Get-ADDomainController -Filter "*"
  $users = Get-ADUser -Filter *

  foreach($user in $users)
  {
    $time = 0
    foreach($dc in $dcs)
    { 
      $hostname = $dc.HostName
      $currentUser = Get-ADUser $user.SamAccountName | Get-ADObject -Server $hostname -Properties lastLogon

      if($currentUser.LastLogon -gt $time) 
      {
        $time = $currentUser.LastLogon
      }
    }
    $dt = [DateTime]::FromFileTime($time)   
    @{
      name = $user.SamAccountName
      lastlogon = $time
      }
  }
}
```

On our test Active Directory, the result will take roughly 9 hours as shown here:

[![GetADUser-MeasureCommand-Attempt1](http://dimitri.janczak.net/wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt1.png)](http://dimitri.janczak.net/wp-content/uploads/2015/07/GetADUser-MeasureCommand-Attempt1.png)

Final remark before starting optimizing our code: some of you may wonder what is this LastLogonDate attribute in Powershell, it won’t help solve our work as you can [there](http://social.technet.microsoft.com/wiki/contents/articles/22461.understanding-the-ad-account-attributes-lastlogon-lastlogontimestamp-and-lastlogondate.aspx).