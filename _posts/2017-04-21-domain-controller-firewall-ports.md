---
id: 715
title: 'domain controller firewall ports'
date: '2017-04-21T14:43:37+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=715'
permalink: /2017/04/21/domain-controller-firewall-ports/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"717";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/04/Windows-Server-Domain-Controller-Replication-Ports-For-Firewall.png
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - DFS-R
    - DMZ
    - domain
    - firewall
    - FRS
    - NetLogon
    - ntds
    - ports
    - replication
---

Often sought on the Internet, rarely complete, here is for a domain controller firewall ports to open so your Windows domain-controller is able to contact the other domain controllers it is depending on for proper replication

- UDP/123 for time synchronization, as in a domain by default the W32Time of a domain controller synchronizes with other domain controllers or the PDCE FSMO role of the top domain of the forest
- TCP/464 and UDP/464 for joining and regularly changing passwords
- TCP/445 for SMB communication (forget about 137, 138, they are unnecessary since Windows 2000!)
- TCP/88 and UDP/88 for Kerberos communication (although you can [force Kerberos to use TCP](https://support.microsoft.com/en-us/kb/244474/en-us) if you wish)
- TCP and UDP/53 for DNS resolution
- TCP/389 and UDP/389 for LDAP
- TCP/636 if you are using LDAPS
- TCP/3268 as global catalog
- TCP/3269 as global catalog over SSL/TLS
- TCP/135 for the RPC endpoint mapper
- a range of ports, by default, 49152-65535 for RPC dynamic ports; you can (and should) [limit them so the RPC ports use a narrower range of ports](https://support.microsoft.com/en-us/kb/929851/en-us). The number of ports depend on the workload of the machine. Thousand ports is more than OK in most scenarios.
- TCP/5722 on Windows 2008(R2) if you use DFS-R to replicate SYSVOL. Due to a bug under that specific version you cannot change that port. On other versions, it is part of the dynamic port range or is set to a specific port if you use the appropriate dfsrdiag starticrpc /port:nnnnn /member:&lt;nameoftheserver&gt;
- the NetLogon and NTDS ports which are part of the dynamic port range unless you use

As a bonus for this post, here is a nice poster for you to dream about that:  
[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-Domain-Controller-Replication-Ports-For-Firewall.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-Domain-Controller-Replication-Ports-For-Firewall.png)IIn addition to domain controller firewall ports, you may need a list of [member server firewall ports](https://dimitri.janczak.net/2015/05/22/member-server-firewall-ports/), as in that case there are less ports to open.