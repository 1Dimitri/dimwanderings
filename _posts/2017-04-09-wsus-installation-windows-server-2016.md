---
id: 702
title: 'WSUS installation on Windows Server 2016'
date: '2017-04-09T16:15:10+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=702'
permalink: /2017/04/09/wsus-installation-windows-server-2016/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"697";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS.png
categories:
    - 'WSUS (Windows Server Update Services)'
tags:
    - 'Add Roles and Features'
    - GUI
    - screenshot
    - 'server manager'
    - 'Windows Server 2016'
---

This post like the IIS default role services under Windows 2012 R2, is a graphical walkthrough of the WSUS installation under Windows 2016. The screenshoots can be used to quickly illustrate some documentation.

# Standard Add Roles And Features Wizard (Windows 2016 look)

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Welcome-Screen.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Welcome-Screen.png)

# Add Roles and Features Wizard Installation type selection screen [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Select-Installation-Type.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Select-Installation-Type.png)

# Add Roles and Features Wizard “Select Server Roles” screen[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Select-WSUS.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Select-WSUS.png)

In this particular case, we only select the Windows Server Update Services’ to perform the WSUS installation on Windows Server 2016

# Add Roles and Features Wizard “Select Features” screen

# [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Select-Feature-WID.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-Select-Feature-WID.png)

# Add Roles and Features Wizard “Windows Server Update Services” configuration screen

# [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS.png)

# Add Roles and Features Wizard “Select Role services” for WSUS installation screen

In this particular screen, you’re able to choose between the WIndows Internal Database (WID Connectivity) and a regular SQL Server (SQL Server Connectivity) to host the database back-end.

# [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Role-Services.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Role-Services.png)

# Add Roles and Features Wizard “Content location” screen

In this screen, you’ll select if you will also host the patches locally or just make the clients refer to the Microsoft Update Internet site. If you choose to host updates locally, you must give a server-accessible path.

# [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Content-Location.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Content-Location.png)

Here I choose to call the directory ‘WSUSContents’. You may want to avoid spaces and other diacritics sign as there are many tools including some API out there which allow you to play with that location, but as usually, they may all have issues in a subtle way as soon as the path is not made solely of digits and letters.[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Content-Location-Filled.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Content-Location-Filled.png)

# Add Roles and Features Wizard “Confirm installation selections” screen

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Confirm.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Confirm.png)

# Add Roles and Features Wizard “Installation progress” screen

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Installation-Progress.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Installation-Progress.png)

# Add Roles and Features Wizard “Installation finished” screen

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Installation-Finished.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/Windows-Server-2016-Add-Roles-And-Features-Wizard-WSUS-Installation-Finished.png)

The remaining screens are published in the [WSUS Configuration on Windows Server 2016](https://dimitri.janczak.net/2017/04/09/wsus-configuration-windows-server-2016/) article.