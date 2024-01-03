---
id: 182
title: 'member server firewall ports'
date: '2015-05-22T19:59:41+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=182'
permalink: /2015/05/22/member-server-firewall-ports/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"188";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/05/DMZ-Member-Servers-Authentication-Against-DCs.png
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - DMZ
    - domain
    - firewall
    - ports
---

Often sought on the Internet, rarely complete, here is for member server firewall ports to open for your Windows domain-joined or soon-to-be-joined machine to be able to contact the domain controllers it is depending on:

- UDP/123 for time synchronization, as in a domain by default the W32Time of a member server synchronizes with the Domain Controllers
- TCP/464 and UDP/464 for joining and regularly changing the computer account password
- TCP/445 for SMB communication (forget about 137, 138, they are unnecessary since Windows 2000!)
- TCP/88 and UDP/88 for Kerberos communication (although you can [force Kerberos to use TCP](https://support.microsoft.com/en-us/kb/244474/en-us) if you wish)
- TCP and UDP/53 for DNS resolution
- TCP/389 and UDP/389 for LDAP
- TCP/636 if you are using LDAPS
- TCP/3268 for joining the domain controllers as global catalog
- TCP/3269 for joining the domain controllers as global catalog over SSL/TLS
- TCP/135 for the RPC endpoint mapper
- a range of ports, by default, 49152-65535 for RPC dynamic ports; you can (and should) [limit them so the RPC ports use a narrower range of ports](https://support.microsoft.com/en-us/kb/929851/en-us). The number of ports depend on the workload of the machine. Thousand ports is more than OK in most scenarios.

As a bonus for this post, here is a nice poster for you to dream about that:[![DMZ-Member-Servers-Authentication-Against-DCs](http://dimitri.janczak.net/wp-content/uploads/2015/05/DMZ-Member-Servers-Authentication-Against-DCs.png)](http://dimitri.janczak.net/wp-content/uploads/2015/05/DMZ-Member-Servers-Authentication-Against-DCs.png)In addition to the member server firewall ports, you may need the [domain controller firewall ports](https://dimitri.janczak.net/2017/04/21/domain-controller-firewall-ports/) list