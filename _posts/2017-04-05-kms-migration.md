---
id: 666
title: 'KMS migration with OS, Office hosting'
date: '2017-04-05T16:17:56+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=666'
permalink: /2017/04/05/kms-migration/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"671";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Welcome-Screen.png
categories:
    - Licensing
tags:
    - 'Key Management'
    - KMS
    - Office
    - screenshot
    - slmgr
---

This post will give real-examples of a KMS migration: adding keys for the OS, Office products to a new machine while removing them from older machines. This post doesn’t speak about AD activated products.

While in Windows 2008 R2, you were relying mostly on [slmgr.vbs](https://technet.microsoft.com/en-us/library/dn502540(v=ws.11).aspx), the Volume Activation Services wizard in Windows 2012R2 allows you to do many operations graphically.

# Steps of the KMS Migration

THe KMS migration is conceptually very simple:

- get a list of all products installed on the current KMS Servers
- for every installed product, install the key on the new server
- for every installed product, remove the key on the old server
- set back the old server as a KMS client

# What’s an Activation ID ?

You may be puzzled by the Activation ID appearing multiple times in the documentation of the slmgr.vbs command. Activation ID is nothing else than a Product ID. You will get one for the OS, one for Office 2010, one for Office 2013, one for Office 2016… The only difficulty here is that the OS key is the “default” activation ID, and that newer OS keys also activate previous versions of the OS, so you don’t need multiple OS keys as opposed as to Office products. Please also note that the Office ProPlus keys also activate the Visio product.

For Office, those Activation IDs are added whenever you install the various Office Key Management Host / Volume License Pack software you can find in the volume license center:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Office-2016-Volume-License-Pack-Installation.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Office-2016-Volume-License-Pack-Installation.png)For OSes, KMS versions are backward-compatible, i.e. Windows 2012R2 will have no issue with Windows 2012(R2) keys whereas Windows 2008R2 needs a hotfix to be able to handle such keys. You may want to search for [KB2885698 – Update adds support for Windows 8.1 and Windows Server 2012 R2 clients to Windows Server 2008, Windows 7, Windows Server 2008 R2, Windows 8, and Windows Server 2012 KMS hosts](https://support.microsoft.com/en-us/help/2885698/update-adds-support-for-windows-8.1-and-windows-server-2012-r2-clients-to-windows-server-2008,-windows-7,-windows-server-2008-r2,-windows-8,-and-windows-server-2012-kms-hosts) for example

# Installing KMS keys on new servers

- In an elevated prompt, you may want to run: ```
    slmgr /ipk <Key>
    ```
    
    If you are using a proxy to go through the Internet, do not forget to setup the WinHTTP proxy through the
    
    ```
    netsh winhttp set proxy command
    ```
    
    for example. Just use the newest key you have; meaning if you have a Windows 2016 Server, it will also activate Windows 2012R2, 2012, 2008R2 and 2008 servers. In addition, server keys also activate Professional and Enterprise client OSes.  
    On Windows 2012(R2) this is equivalent to this page of the wizard:[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Welcome-Screen.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Welcome-Screen.png) [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Enter-KMS-Key-Screen.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Enter-KMS-Key-Screen.png)
- Once the key is installed, you may want to activate the product: ```
    slmgr /ato
    ```
    
    [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Activate-Prompt-Screen.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Activate-Prompt-Screen.png)
- For every Office product, you will have to use the same command altogether with the Activation ID: ```
    slmgr /ipk <Key> ActivationID
    ```
    
    The screen would be:[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Activate-Product-Screen.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-Activate-Product-Screen.png)
- Do not forget to activate each key: ```
    slmgr /ato ActivationID
    ```

# Checking the service is operating correctly

- Check the TCP/1688 is open in the Windows Advanced Firewall[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-Service-Advanced-Firewall-Rule-Enabled.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-Service-Advanced-Firewall-Rule-Enabled.png)
- Check the server is publishing correctly in the Active Directory if you don’t want to manually input the new KMS server name into every client
- Do not forget that the service will start to deliver new licenses once the minimum of clients requesting such licenses is reached. Thresholds are different for OS and Office Products. Otherwise you’ll get “0xC004F038: The software Licensing service reported that the computer could not be activated” error.[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-Service-Not-Enough-Clients-Yet.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-Service-Not-Enough-Clients-Yet.png)

# Removing KMS Keys on old servers

1. Start by unpublishing the SRV DNS Record so clients don’t try to connect anymore: ```
    slmgr /cdns
    ```
    
    [![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-Clear-DNS-Publish-Activated-Only.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-Clear-DNS-Publish-Activated-Only.png)
2. Get a list of used keys for every Activation ID by using the Status line of the ```
    slmgr /dli
    ```
    
    output
3. For every activated product run a ```
    slmgr /upk <Key> ActivationID
    ```
4. Once the server is no longer hosting any KMS key, do not forget to make the server a KMS Client client again, by inputting the correct [KMS client key](https://technet.microsoft.com/en-us/library/jj612867(v=ws.11).aspx), using ```
    slmgr /ipk
    ```
5. and re-activate the server by running ```
    slmgr /ato
    ```
    
    again

# KMS Configuration through Wizard

The last page of the Windows 2012(R2) VAMT wizard allows you to perform configuration of the KMS Service itself:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-KMS-Configuration.png)](https://dimitri.janczak.net/wp-content/uploads/2017/04/KMS-VAMT-Volume-Activation-Services-Wizard-KMS-Configuration.png)The options are slmgr /sai, slmgr /sri, slmgr /sprt, slmgr /cdns and slmgr /sdns