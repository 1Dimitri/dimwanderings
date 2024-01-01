---
id: 445
title: 'Error 1753: no more endpoints in AD Replication'
date: '2016-07-06T21:30:42+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=445'
permalink: /2016/07/06/error-1753-active-directory-replication/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"449";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/07/RPC-process.gif
categories:
    - 'Active Directory'
    - 'troubleshooting tips'
tags:
    - 'Active Directory'
    - 'endpoint mapper'
    - EPM
    - RPC
    - troubleshooting
---

When you see Error 1753: no more endpoints in AD Replication, the common mistake is to think about firewall or TCP/IP connectivity issues. The message “Error 1753: There are no more endpoints available from the endpoint mapper” does not mean that you cannot connect at all.

It is quite the opposite: it means that in fact the RPC client could connect to the RPC server using the TCP port 135. But that the RPC server replied with an error. The RPC server is basically telling the client that it could not find an endpoint for the specific service the client requested.

By the “service”, we must understand the (authenticated server,service) pair. It means that we must check that the client is talking to the correct server it thinks it has contacted and that the service exists on that server.

For Active Directory Service, checking that the server is really the one it claims to be means:

- the GUID, IP and the names in the various DNS records for \_msdcs.forest.local are OK. Typically you may encounter issues if you have changed IP addresses before or after promotion and the information has not replicated yet. It can also happen if you swapped Domain Controller IP Addresses, or if you reinstalled a Domain Controller from scratch (Like installing a newer OS version)
- On the DC where you have the 1753 errors, you may want to check:

```
<pre class="">nslookup -type=cname <name of source DC you are not replicating with> <DNS Server Used by the DC you are on>
```

To check that the service is running, check that

- Active Directory Domain Services is running
- you have something which listens on port 135/tcp
- To know if the service has registered its RPC UUID is more complex, the simplest way to do it is to use portqry and query for the Endpoint Mapper by using: ```
    <pre class="">portqry -n <targetIP> -e 135
    ```
- Juniper has a [list](http://kb.juniper.net/InfoCenter/index?page=content&id=KB12057) of the most common UUID you can compare with. The one you’re interested in is MS NT Directory DRS interface