---
id: 782
title: 'Windows 2016 Remote Desktop cannot verify the identity'
date: '2017-10-24T09:21:19+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=782'
permalink: /2017/10/24/windows-2016-remote-desktop-cannot-verify-identity/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"781";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/10/Remote-Desktop-Cannot-Verify-The-Identity-Of-The-Remote-Computer.png
categories:
    - 'troubleshooting tips'
tags:
    - 'Active Directory'
    - DNS
    - GPO
    - PolicSOM
    - RDP
    - 'Remote Desktop'
    - time
    - w32tm
    - 'Windows Server 2016'
    - wmi
    - WMIPRVSE
---

When connecting using RDP to a Windows 2016 server, you may receive an error message which seems clueless to you: Windows 2016 Remote Desktop cannot verify the identity of the remote computer because there is a time or date difference between your computer and the remote computer.

When you manage to connect to the host, using the console or a Powershell session, you immediately [check the time](https://dimitri.janczak.net/2017/02/07/ntpclient-error-0x800706e1/) and you see no discrepancy between that machine and the one you’re connecting from. When you investigate further you notice warning and errors for the DNS Client service stating it could not register the server. The GPO client side fails to be applied because no Domain Controller is to be found. However if you troubleshoot the connectivity between the Domain Controller/DNS Server and your machine, you see no error in the firewall, just that the DNS Service isn’t answering, whereas other clients don’t have any issue. On the machine, you are likely to encounter the event 8015 of source DNS Client Events. “The system failed to register host (A or AAAA) resource records (RRs) for network adapter”

[![](https://dimitri.janczak.net/wp-content/uploads/2017/10/DNS-Client-Event-8015.png)](https://dimitri.janczak.net/wp-content/uploads/2017/10/DNS-Client-Event-8015.png)

If you look carefully at the event viewer, you’ll see however an error from the TCP/IP stack, saying that there is no UDP Ports left. If you issue an netstat -a -b -n command, you’ll see they are all eaten up by the WMIPrvSe.exe process. KIlling this process doesn’t help. If you restart the machine everything is of course cleared up. The message (Source: TCPIP, event 4266) you get before the DNS issues is “A request to allocate an ephemeral port number from the global UDP port space has failed due to all such ports being in use”.

[![](https://dimitri.janczak.net/wp-content/uploads/2017/10/TcpIp-Event-4266.png)](https://dimitri.janczak.net/wp-content/uploads/2017/10/TcpIp-Event-4266.png)

If you are able to force [Kerberos over TCP](https://support.microsoft.com/en-us/help/244474/how-to-force-kerberos-to-use-tcp-instead-of-udp-in-windows) before the next issue, you would see that you don’t have trouble to connect, but that the UDP port exhaustion is still present. But how come that the WMI subsystem eats all those ports? The clue is not on the machine but in the domain.

According to the Microsoft Premier Support, there is a bug under Windows 2016: when applying WMI Filtered GPOs, UDP Ports are not released enough by the [Policy provide](https://msdn.microsoft.com/en-us/library/aa392744(v=vs.85).aspx)r when using the WMIPRVSE process. Then you’re ending up in this situation. A work-around is to have other services relying on TCP (such as Kerberos with the proper registry entry), but a better work-around is to have the WMI process isolated for this consumption.

If you face this issue, the no-reboot solution is then:

1. Connect to the machine using a PSSession
2. Execute the following code to isolate the provider in its own process: ```
    <pre class="lang:default decode:true" title="Giving the PolicMan Provider its own WMIPRVSE process">$a = [WMI]'root\policy:__Win32Provider.name="PolicSOM"'
    $a.HostingModel = "NetworkServiceHost:Debug1"
    $a.put()
    ```
3. Restart the WinMgmt service (Restart-Service WinMgmt -Force will do the trick as there are dependencies)
4. No need to reboot to re-issue a RDP connection
5. If you want to revert back to test the behavior, issue the following snippet: ```
    <pre class="lang:default decode:true" title="Revert Policy Manager process to NetworkService host thing">$a = [WMI]'root\policy:__Win32Provider.name="PolicSOM"'
    $a.HostingModel = "NetworkServiceHost"
    $a.put()
    ```