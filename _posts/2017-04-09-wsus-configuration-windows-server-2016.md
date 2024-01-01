---
id: 705
title: 'WSUS configuration on Windows Server 2016'
date: '2017-04-09T16:19:26+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=705'
permalink: /2017/04/09/wsus-configuration-windows-server-2016/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"687";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Welcome-Screen.png
categories:
    - 'WSUS (Windows Server Update Services)'
tags:
    - 'Add Roles and Features'
    - GUI
    - screenshot
    - 'server manager'
    - 'Windows Server 2016'
---

After having performed the [WSUS Installation on Windows Server 2016](https://dimitri.janczak.net/2017/04/09/wsus-installation-windows-server-2016/), let’s continue with the WSUS Configuration on Windows Server 2016 screenshots.

# WSUS Wizard : Store updates screen

You may notice that this information has already been asked in a similar fashion while performing the [WSUS Installation on Windows Server 2016](https://dimitri.janczak.net/2017/04/09/wsus-installation-windows-server-2016/).

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Wizard-Location.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Wizard-Location.png)

# WSUS Wizard : installation complete screen

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Wizard-Complete.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Wizard-Complete.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Wizard-Progress.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Wizard-Progress.png)

# WSUS Configuration Wizard : Welcome screen

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Welcome-Screen.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Welcome-Screen.png)

# WSUS Configuration Wizard : CEIP screen

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-CEIP.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-CEIP.png)

# WSUS Configuration Wizard : Upstream/Downstream server screen

You may notice that the wording only mentions upstream server for the updates location. it doesn’t explicitly tell you if your server is going to be an [upstream or downstream server](https://technet.microsoft.com/en-us/library/cc708495(v=ws.10).aspx) but speaks of ‘replica of an upstream server’

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Synchronisation-Source.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Synchronisation-Source.png)

# WSUS Configuration Wizard: proxy for upstream server screen[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Proxy-Choice.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Proxy-Choice.png)

# WSUS Wizard : attempt to connect to Microsoft update screen[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Attempt-To-Connect.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Attempt-To-Connect.png)

# WSUS Wizard: exception while connecting to Microsoft Update dialog box[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Connection-TimeOut.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Connection-TimeOut.png)

# WSUS Wizard: connection to Microsoft update summary screen[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Connection-TimeOut-Wizard-Error.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-WSUS-Role-Wizard-Connection-TimeOut-Wizard-Error.png)

Note that in the latest screenshots, the WSUS server is unable to downloaded patches from the Internet as the proxy is configured not to allow that machine. In that case, you’ll receive an .NET based HTTP exception and an error at the end of the wizard.