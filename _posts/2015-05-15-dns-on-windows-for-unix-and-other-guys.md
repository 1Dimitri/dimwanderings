---
id: 149
title: 'DNS Servers on Windows for Unix (and other) guys.'
date: '2015-05-15T21:05:22+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=149'
permalink: /2015/05/15/dns-on-windows-for-unix-and-other-guys/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"151";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/05/dns.jpg
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - bind
    - daemon
    - DNS
    - dnscmd
    - unix
---

I’m a happy reader of[ Stephane Bortzmeyer’s blog](http://www.bortzmeyer.org/). He does a good job of presenting and analyzing RFCs you wouldn’t often otherwise have time to read. The only thing that bothers me is that Stephane -like many of my colleagues- has no deep knowledge of the [Windows Server counterparts](http://www.bortzmeyer.org/vider-cache-resolveur.html) of his huge Unix admin skills, so I must always adapt his demonstrations to the professional world I live in 😉

In particular, When it comes to the DNS daemons, Unix people tend to be lost in translation. So here is a list of a few useful things to know to operate DNS Servers on Windows

- Windows DNS Service is available on… Windows Servers, not Windows Clients 
    - There are other DNS Services implementations running on the Microsoft platform however
    - If you want to test this implementation or have to troubleshoot interoperability issues, 6-month trials can be downloaded at Microsoft’s
- Any Windows server can host the DNS Service (daemon for you unix guys), but this service is often found on Domain Controllers as Active Directory heavily rely on A and SRV record for communication between clients and Domain Controllers, or between Domain Controllers
- Windows DNS Servers can act as primary or secondary for zones and can interact with non Windows servers if RFCs are followed
- Windows DNS Servers can read DNS zones files
- The tools to manage the DNS zones and servers are 
    - The graphical MMC dns.msc including [some troubleshooting and logging features](http://dimitri.janczak.net/2015/05/15/dns-on-windows-for-unix-and-other-guys/)
    - The command-line tool dnscmd.exe, In the Resource Toolkit for Windows 2003, and built-in with Windows 2008/vista or higher (also available in the Remote Server Administration Tools for the client OS)
    - The WMI provider class is root\\MicrosoftDNS
    - Since Windows 2012R2, powershell cmdlets
    - nslookup.exe is available on every Windows OS since Windows 2008/Vista but there’s no built-in dig. However, debugging queries can be achieved by enable verbose logging: you’ll get the standard Query/Response message you’re familiar with.
- Albeit not directly related to DNS, Microsoft Message Analyzer, successor of the Microsoft Network Monitor, is able to capture network traces and link them with running processes and even to save those traces as .cap files if your colleagues are only familiar with WireShark or tcpdump 😉
- There’s an “Active Directory Integrated” Zone mode which is very Microsoft-specific: 
    1. All Windows Servers that host that zone act as primary for the zone
    2. You can set-up ACLs (Access Control List) on every record of the zone, such as granting some Windows groups, users or computers the right to read, write those records
    3. You can choose on which servers the zone will be hosted: DNS Servers of a given domain, Domain Controllers of the domain or of the forest.

From all the above, the only thing that may raise an eyebrow to an Unix sysadmin is this “Active Directory Integrated Zone”. It is easy however to understand if you know that

1. such a zone is no longer hosted as a zone file but as DNSzone entries in the LDAP database directory that the Active Directory is in fact
2. Active Directory is as ever a multi-master replicated database
3. You can put any Windows Security Identifier on such Active Directory Objects
4. you can choose to store those entries in different partitions (different namespaces) of the Active Directory

(1) is a result of (a) and (b), (2) is a result of (c) and (3) is a result of (a), (b) and (d).

Hope this post will allow a few sysadmins from both worlds to speak the same language, and there’s a[ few images](http://dimitri.janczak.net/2015/05/20/windows-dns-server-graphical-troubleshooting-tools/) for the ones who are still not scared.