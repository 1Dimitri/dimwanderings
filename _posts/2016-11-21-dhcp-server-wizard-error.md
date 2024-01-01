---
id: 539
title: 'DHCP Server Wizard Error'
date: '2016-11-21T17:08:49+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=539'
permalink: /2016/11/21/dhcp-server-wizard-error/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"543";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/11/DHCP-Server-Role-In-Server-Manager.png
categories:
    - 'DHCP Server'
tags:
    - installation
    - 'server manager'
    - troubleshooting
---

When you [install the DHCP Server Role](https://blogs.technet.microsoft.com/teamdhcp/2012/08/31/installing-and-configuring-dhcp-role-on-windows-server-2012/) on Windows 2012 R2, you may not be able to finish the post-installation wizard successfully. This DHCP Server Wizard Error message starts with “Failed to open registry key on target computer to set the status of post configuration task.”

And every tine you reboot the server you get the prompt:

[![Complete DHCP Configuration wizard window](https://dimitri.janczak.net/wp-content/uploads/2016/11/Task-Complete-DHCP-Configuration.png)](https://dimitri.janczak.net/wp-content/uploads/2016/11/Task-Complete-DHCP-Configuration.png)  
Various messages can be:

- Error: The WinRM client received an HTTP status code of 504 from the remote WS-Management service
- Error: The computer name cannot be resolved

To get rid of the message, the registry key to fix this is under the key HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\ServerManager\\Roles\\12. The value name is ConfigurationState and it must be changed from 1 to 2.

You must perform this operation when you are sure the operations the wizard would do are OK:

- If the server is member of a domain, the server is authorized in that domain. If you have re-used, re-installed the server, this may be the cause of your error
- The local group DHCP Users and DHCP Administrators exist on the server

If you’re unsure about the tasks that are OK, go to the last page of the wizard by committing the changes it proposes. You should get a image such as

[![DHCP Server Post Installation Status](https://dimitri.janczak.net/wp-content/uploads/2016/11/DHCP-Server-Post-Installation-Wizard-Status.png)](https://dimitri.janczak.net/wp-content/uploads/2016/11/DHCP-Server-Post-Installation-Wizard-Status.png)Here the security groups were created, but the authorization was already given in a previous attempt to run the installation process.

If you modify settings a restart of the DHCP Serever service may be needed.