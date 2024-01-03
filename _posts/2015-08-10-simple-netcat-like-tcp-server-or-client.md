---
id: 339
title: 'Simple netcat-like TCP Server or client'
date: '2015-08-10T13:08:26+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=339'
permalink: /2015/08/10/simple-netcat-like-tcp-server-or-client/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"340";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/08/firewall-and-internet-as-earth.png
categories:
    - 'Powershell Engine'
tags:
    - firewall
    - ncat
    - netcat
---

Need to have a process listening on some port to test your firewall rules? Instead of using a netcat or ncat, here’s a very simple Powershell implementation of this ncat/netcat feature:

```
<pre class="lang:ps decode:true  " title="Simple TCP Server">function Wait-OnPort ($port) {
$endpoint = new-object System.Net.IPEndPoint ([ipaddress]::any,
$port)
$listener = new-object System.Net.Sockets.TcpListener $endpoint
$listener.start()
$listener.AcceptTcpClient() # wait until somebody connect
$listener.stop()
}
```

If you need a TCP Client, the code is very straightforward:

```
<pre class="lang:ps mark:3-7 decode:true" title="Use of TcpClient">$t = New-Object System.Net.Sockets.TcpClient("servernameorIPAddress",port)
#port is not open, you get an exception
New-Object : Exception calling ".ctor" with "2" argument(s): "No connection could be made because the target machine actively refused it 10.254.212.81:666"
At line:1 char:16
+ $t = New-Object <<<<  System.Net.Sockets.TcpClient("servernameorIPAddress",port)
    + CategoryInfo          : InvalidOperation: (:) [New-Object], MethodInvocationException
    + FullyQualifiedErrorId : ConstructorInvokedThrowException,Microsoft.PowerShell.Commands.NewObjectCommand

#port is open, you get nothing
$t.Close()
```