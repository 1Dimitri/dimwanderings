---
id: 549
title: 'DHCP Unauthorize: There is no such object on the server'
date: '2016-12-19T16:40:37+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=549'
permalink: /2016/12/19/dhcp-unauthorize-there-is-no-such-object-on-the-server/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"556";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/12/dhcpClass-object.png
categories:
    - 'DHCP Server'
tags:
    - 'Active Directory'
    - netsh
    - powershell
    - troubleshooting
---

In this blog post, let’s see what you should do if you get during DHCP Unauthorize: There is no such object on the server error message,

# DHCP Authorize / Unauthorize

As you probably know, a DHCP Server must be authorized in an Active Directory to start delivering addresses. This is to avoid rogue servers everybody could set up. The reverse operation is called “unauthorize” and you’re supposed to do it before removing the DHCP role on a server.

In most cases, this is never done as the role is not uninstalled but the server is switched off instead. In this case the old server still appears in the Active Directory as authorized.

# Unauthorizing a server properly

The proper way to do it is to use the [MMC](https://technet.microsoft.com/en-us/library/dd183610(v=ws.10).aspx), the netsh commands or a powershell cmdlet.

For MMC:

- Open the DHCP MMC and click DHCP.
- On the Action menu, click Manage authorized servers
- In the Authorized DHCP servers dialog box, select the server you want to unauthorize.
- Click Unauthorize.

For netsh:

- Check the server list with “netsh dhcp show server”[![netsh show dhcp server](https://dimitri.janczak.net/wp-content/uploads/2016/12/netsh-show-dhcp-server.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/netsh-show-dhcp-server.png)
- Delete each server with “netsh dhcp delete server servername.domain.local IPAddress”[![netsh delete dhcp server](https://dimitri.janczak.net/wp-content/uploads/2016/12/netsh-delete-dhcp-server.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/netsh-delete-dhcp-server.png)

In powershell the cmdlet is called, “Remove-DhcpServerInDC” and not unauthorize-something as the verb is not generic enough:

[![remove-dhcpserverindc](https://dimitri.janczak.net/wp-content/uploads/2016/12/Remove-DhcpServerInDc.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/Remove-DhcpServerInDc.png)The same parameters should be used: DNS Name and IP Address.

# How does it work

The authorization in the Active Directory just creates entries into the NetServices container of your configuration partition of your forest.

IF every goes well you’ve got an object representing for each server you have authorized.

# Troubleshooting issues with unauthorize

If you are somewhere stuck into the middle of the process, you will get messages such as:

- There is no such object on the server
- The parameter is incorrect

In all cases:

- check the following under the CN=”NetServices,CN=Services,CN=Configuration,DC=myforest,DC=local”

[![NetServices Services Configuration container](https://dimitri.janczak.net/wp-content/uploads/2016/12/NetServices-Container.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/NetServices-Container.png)

- You no longer have an object with the name of the server under this container. If the object is still there you may want to delete it after having written down the contents of its dhcpServers attribute.[![dhcpclass-object](https://dimitri.janczak.net/wp-content/uploads/2016/12/dhcpClass-object.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/dhcpClass-object.png)
- There is a DHCPRoot object which has weird entries into its DHCPServers attribute value corresponding to the attributes you saw in the previous step.

[![dhcpRoot object](https://dimitri.janczak.net/wp-content/uploads/2016/12/dhcpServers-attribute-dhcpRoot-object.png)](https://dimitri.janczak.net/wp-content/uploads/2016/12/dhcpServers-attribute-dhcpRoot-object.png)You may want to delete them using ADSIEdit.msc or any tool of your choice.