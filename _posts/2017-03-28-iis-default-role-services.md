---
id: 634
title: 'IIS default role services on Windows 2012 R2'
date: '2017-03-28T15:12:51+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=634'
permalink: /2017/03/28/iis-default-role-services/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"651";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-5.png
categories:
    - IIS
tags:
    - 'IIS 8.5'
    - reference
    - 'Role Services'
    - screenshot
    - 'Windows 2012 R2'
---

This post is aimed at listing what are the default IIS default role services so when a consultant tells you (s)he wants IIS installed but is unable to specify which role services, you can easily document the role services by copy pasting the following screenshots:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-1.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-1.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-2.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-2.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-3.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-3.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-4.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-4.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-5.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-5.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-6.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-6.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-7.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-7.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-8.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-8.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-9.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-9.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-10.png)](https://dimitri.janczak.net/wp-content/uploads/2017/03/Windows-2012R2-IIS-Default-Role-Services-Installation-Wizard-Page-10.png)As you can see on the different screens, the following IIS default role services on Windows 2012 R2 are:

- Common HTTP features 
    - Default document
    - Directory Browsing
    - HTTP Errors
    - Static Content
- Health and Diagnostics 
    - HTTP Logging
- Performance 
    - Static Content Compression
- Security 
    - Request Filtering
- Management Tools 
    - IIS Console

Please note that you can always get the list of installed role services by using the [Get-WindowsFeature](https://msdn.microsoft.com/en-us/library/ee662312.aspx) cmdlet under Powershell. To get everything under the IIS role, you must use Web and not IIS as as keyword

```
PS C:\Windows\system32> Get-WindowsFeature  |  ? {$_.Name -like 'Web*' } | Select DisplayName, Name

DisplayName                                   Name
-----------                                   ----
Web Application Proxy                         Web-Application-Proxy
Web Server (IIS)                              Web-Server
Web Server                                    Web-WebServer
Common HTTP Features                          Web-Common-Http
Default Document                              Web-Default-Doc
Directory Browsing                            Web-Dir-Browsing
HTTP Errors                                   Web-Http-Errors
Static Content                                Web-Static-Content
HTTP Redirection                              Web-Http-Redirect
WebDAV Publishing                             Web-DAV-Publishing
Health and Diagnostics                        Web-Health
HTTP Logging                                  Web-Http-Logging
Custom Logging                                Web-Custom-Logging
Logging Tools                                 Web-Log-Libraries
ODBC Logging                                  Web-ODBC-Logging
Request Monitor                               Web-Request-Monitor
Tracing                                       Web-Http-Tracing
Performance                                   Web-Performance
Static Content Compression                    Web-Stat-Compression
Dynamic Content Compression                   Web-Dyn-Compression
Security                                      Web-Security
Request Filtering                             Web-Filtering
Basic Authentication                          Web-Basic-Auth
Centralized SSL Certificate Support           Web-CertProvider
Client Certificate Mapping Authentication     Web-Client-Auth
Digest Authentication                         Web-Digest-Auth
IIS Client Certificate Mapping Authentication Web-Cert-Auth
IP and Domain Restrictions                    Web-IP-Security
URL Authorization                             Web-Url-Auth
Windows Authentication                        Web-Windows-Auth
Application Development                       Web-App-Dev
.NET Extensibility 3.5                        Web-Net-Ext
.NET Extensibility 4.5                        Web-Net-Ext45
Application Initialization                    Web-AppInit
ASP                                           Web-ASP
ASP.NET 3.5                                   Web-Asp-Net
ASP.NET 4.5                                   Web-Asp-Net45
CGI                                           Web-CGI
ISAPI Extensions                              Web-ISAPI-Ext
ISAPI Filters                                 Web-ISAPI-Filter
Server Side Includes                          Web-Includes
WebSocket Protocol                            Web-WebSockets
FTP Server                                    Web-Ftp-Server
FTP Service                                   Web-Ftp-Service
FTP Extensibility                             Web-Ftp-Ext
Management Tools                              Web-Mgmt-Tools
IIS Management Console                        Web-Mgmt-Console
IIS 6 Management Compatibility                Web-Mgmt-Compat
IIS 6 Metabase Compatibility                  Web-Metabase
IIS 6 Management Console                      Web-Lgcy-Mgmt-Console
IIS 6 Scripting Tools                         Web-Lgcy-Scripting
IIS 6 WMI Compatibility                       Web-WMI
IIS Management Scripts and Tools              Web-Scripting-Tools
Management Service                            Web-Mgmt-Service
IIS Hostable Web Core                         Web-WHC
```

Once the various IIS default role services are installed, you can refer to the [IIS.net configuration reference](https://www.iis.net/configreference) for the XML supported keywords.