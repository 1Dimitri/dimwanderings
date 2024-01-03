---
id: 176
title: 'Windows DNS Server graphical troubleshooting tools'
date: '2015-05-20T19:45:35+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=176'
permalink: /2015/05/20/windows-dns-server-graphical-troubleshooting-tools/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:177;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - 'Windows OS'
tags:
    - bind
    - DNS
    - logging
    - troubleshooting
---

This post is somehow a sequel to the [DNS on Windows for Unix (and other) guys](http://dimitri.janczak.net/2015/05/15/dns-on-windows-for-unix-and-other-guys/).

Using the DNS Management Console, you already have a set of useful tabs for troubleshooting purposes:

- Monitoring also you to perform simple test queries:

[![Windows-DNSServer-Properties-Monitoring](http://dimitri.janczak.net/wp-content/uploads/2015/05/Windows-DNSServer-Properties-Monitoring-259x300.png)](http://dimitri.janczak.net/wp-content/uploads/2015/05/Windows-DNSServer-Properties-Monitoring.png)

- Debug Logging is more interesting as it works as a dedicated network packet capture tool: 
    - Please note that there’s no browse button to fill in the ‘File Path and name’ so you have to type the path (I choose to put that into C:\\Windows\\temp but you can put it elsewhere as long as the DNS Server service can write to that location)
    
    [![Windows-DNSServer-Properties-DebugLogging](http://dimitri.janczak.net/wp-content/uploads/2015/05/Windows-DNSServer-Properties-DebugLogging1-262x300.png)](http://dimitri.janczak.net/wp-content/uploads/2015/05/Windows-DNSServer-Properties-DebugLogging1.png)
    
    
    - You then obtain data like this excerp: ```
        <pre class="lang:default decode:true" title="DNS Log file on Windows">DNS Server log file creation at 5/21/2015 1:51:18 PM
        Log file wrap at 5/21/2015 1:51:18 PM
        
        Message logging key (for packets - other items use a subset of these fields):
        	Field #  Information         Values
        	-------  -----------         ------
        	   1     Date
        	   2     Time
        	   3     Thread ID
        	   4     Context
        	   5     Internal packet identifier
        	   6     UDP/TCP indicator
        	   7     Send/Receive indicator
        	   8     Remote IP
        	   9     Xid (hex)
        	  10     Query/Response      R = Response
        	                             blank = Query
        	  11     Opcode              Q = Standard Query
        	                             N = Notify
        	                             U = Update
        	                             ? = Unknown
        	  12     [ Flags (hex)
        	  13     Flags (char codes)  A = Authoritative Answer
        	                             T = Truncated Response
        	                             D = Recursion Desired
        	                             R = Recursion Available
        	  14     ResponseCode ]
        	  15     Question Type
        	  16     Question Name
        
        5/21/2015 1:51:18 PM 0D48 PACKET  000000000181D1C0 UDP Rcv 10.250.2.246    0c6b   Q [0001   D   NOERROR] A      (10)sitecheck2(5)opera(3)com(0)
        5/21/2015 1:51:18 PM 0D48 PACKET  000000000509A200 UDP Snd 10.250.128.151  b94d   Q [0001   D   NOERROR] A      (10)sitecheck2(5)opera(3)com(0)
        5/21/2015 1:51:18 PM 0D48 PACKET  00000000045FDBF0 UDP Rcv 10.250.128.151  b94d R Q [8081   DR  NOERROR] A      (10)sitecheck2(5)opera(3)com(0)
        5/21/2015 1:51:18 PM 0D48 PACKET  000000000181D1C0 UDP Snd 10.250.2.246    0c6b R Q [8081   DR  NOERROR] A      (10)sitecheck2(5)opera(3)com(0)
        5/21/2015 1:51:18 PM 0D48 PACKET  0000000003FBFAE0 UDP Rcv 10.250.2.246    1425   Q [0001   D   NOERROR] A      (5)ctldl(13)windowsupdate(3)com(0)
        5/21/2015 1:51:18 PM 0D48 PACKET  000000000181D1C0 UDP Snd 10.250.128.151  b866   Q [0001   D   NOERROR] A      (5)a1621(1)g(6)akamai(3)net(0)
        5/21/2015 1:51:18 PM 0D44 PACKET  00000000045FDBF0 UDP Rcv 10.250.2.246    c556   Q [0001   D   NOERROR] A      (5)ctldl(13)windowsupdate(3)com(0)
        5/21/2015 1:51:18 PM 0D44 PACKET  000000000534C850 UDP Rcv 10.250.128.151  b866 R Q [8081   DR  NOERROR] A      (5)a1621(1)g(6)akamai(3)net(0)
        5/21/2015 1:51:18 PM 0D44 PACKET  0000000003FBFAE0 UDP Snd 10.250.2.246    c556 R Q [8081   DR  NOERROR] A      (5)ctldl(13)windowsupdate(3)com(0)
        5/21/2015 1:51:18 PM 0D44 PACKET  0000000003FBFAE0 UDP Snd 10.250.2.246    1425 R Q [8081   DR  NOERROR] A      (5)ctldl(13)windowsupdate(3)com(0)
        5/21/2015 1:51:18 PM 0D44 PACKET  000000000534C850 UDP Rcv 10.250.2.246    10ab   Q [0001   D   NOERROR] A      (10)duckduckgo(3)com(0)
        5/21/2015 1:51:18 PM 0D44 PACKET  0000000003FBFAE0 UDP Snd 10.250.128.151  a7cb   Q [0001   D   NOERROR] A      (10)duckduckgo(3)com(0)
        ...
        ```
        
        As you can see the header is full of information, from the time the log was taken/wrapped (these are local times to the server) to a complete reminder of the various fields you may encounter.