---
id: 106
title: 'Best DFS-R practice against dirty shutdowns'
date: '2015-01-27T15:12:00+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=106'
permalink: /2015/01/27/best-dfs-r-practice-against-dirty-shutdowns/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"108";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2015/01/dfs-r.png
categories:
    - 'Windows OS'
tags:
    - DFS-R
    - 'domain controller'
    - 'Windows 2008 R2'
---

If you have ever played with the DFS-R technology elsewhere than in a lab, i.e. with more than a couple of files or an almost empty sysvol shared by two domain controllers, you have already encountered the ‘dirty shutdown symptom’:

The system at some point finds its database is corrupt and stops the replication waiting for some human to restart the replication.

Acknowledging issues in the code, Microsoft released a lot of hotfixes for this technology in Windows 2008 R2: there was KB2663685, then KB2698155, KB2831101, KB2851868, KB2884176 and now [KB3002288](http://support.microsoft.com/kb/3002288/en-us) at the time of this writing.

The problem does not stop at applying the hotfixes. The main problem is to restart the replication and have your volumes good again quickly.

Depending on how you are fast (or how fast are your change / ticket procedures are), how big is the replicated data volume, you may have to wait for several hours or days for replication to be good again.

One first step achieved by Microsoft is the implementation of the StopReplicationOnAutoRecovery registry introduced with hotfix KB2663685 (or higher). Be sure to set this value to 0, and not 1 as many hotfixes would require.

The second step is to have documented the [existing hotfixes](http://support.microsoft.com/kb/968429/en-us). Be sure to check this page periodically as we had already more than a couple of updates.

The third step is that the [best practices](http://support.microsoft.com/kb/2846759/en-us) by server type are finally shown here, and they change from the Windows 2008 release times. The cherry on the cake would be to have a article title that reflects that it also highlights new recommended settings.

The other solution is to migrate to Windows 2012 R2 according to Microsoft, but as usual, Microsoft forgets we cannot afford to migrate all our servers to the latest version at the same time.