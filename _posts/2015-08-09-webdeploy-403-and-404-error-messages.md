---
id: 331
title: 'WebDeploy 403 and 404 Error Messages'
date: '2015-08-09T22:54:43+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=331'
permalink: /2015/08/09/webdeploy-403-and-404-error-messages/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"333";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/08/webdeploy_35_install_options.png
categories:
    - IIS
tags:
    - msdeploy.axd
    - msdepsvc
    - troubleshooting
    - 'Visual Studio'
    - 'Web Management Service'
    - WebDeploy
    - wmsvc
---

If you use WebDeploy on your IIS Servers, and have [ trouble to publish web sites on them using the WebDeploy tool](http://www.iis.net/learn/publish/troubleshooting-web-deploy/web-deploy-error-codes), you may have received errors such as:  *Could not connect to the remote computer (“targetserver”). On the remote computer, make sure that Web Deploy is installed and that the required process (“Web Management Service”) is started. Learn more at http://go.microsoft.com/fwlink/?LinkId=221672#ERROR\_DESTINATION\_NOT\_REACHABLE.  
Error: The remote server returned an error: (404) Not Found.*

You have probably found[ articles](http://blogs.iis.net/kateroh/troubleshooting-common-msdeploy-issues) saying you must check the firewall on port 8172 and restart services such as MsDepSvc and WmSvc.

However those articles fail to explain that both services do not come with WebDeploy and these workaround don’t solve 40x errors.

- MsDepSvc comes indeed with WebDeploy but doesn’t list’en on the TCP/8172 port; it just adds a .axd HTTP module to perform its work
- This module works on top of the IIS Management Service WmSvc which is the listener on port 8172 and comes as a role service under IIS 7.0 or higher

Therefore if you encounter various error messages during your WebDeploy tasks, check

1. that the IIS Management service has been installed as the MSI Installer for WebDeploy (at least in version up to 3.6) doesn’t check for the prerequisites:[![Web Management Service_role_services_selection](http://dimitri.janczak.net/wp-content/uploads/2015/08/Web-Management-Service_role_services_selection.jpg)](http://dimitri.janczak.net/wp-content/uploads/2015/08/Web-Management-Service_role_services_selection.jpg)
2. Configure this service in the IIS Console to accept remote connections:[![IIS-Web-Management-Service-Configuration](http://dimitri.janczak.net/wp-content/uploads/2015/08/IIS-Web-Management-Service-Configuration.png)](http://dimitri.janczak.net/wp-content/uploads/2015/08/IIS-Web-Management-Service-Configuration.png)
3. re-install or repair WebDeploy using its MSI for everything to work.:[![webdeploy_35_install_options](http://dimitri.janczak.net/wp-content/uploads/2015/08/webdeploy_35_install_options.png)](http://dimitri.janczak.net/wp-content/uploads/2015/08/webdeploy_35_install_options.png)