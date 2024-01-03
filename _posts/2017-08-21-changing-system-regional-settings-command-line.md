---
id: 764
title: 'Changing the System Regional settings after installation using the command-line'
date: '2017-08-21T14:20:47+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=764'
permalink: /2017/08/21/changing-system-regional-settings-command-line/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"770";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/08/Windows-Control-Panel-Clock-Language-Regions.png
categories:
    - Uncategorized
    - 'Windows OS'
tags:
    - 'command line'
    - 'control panel'
    - installation
    - localization
    - 'regional settings'
    - unattended
    - 'user locale'
---

Usually you set up the regional settings at installation time by using localized OS Version or unattended.xml. However sometimes you discover after installation that some script from your customer run under the system context needs different settings. In this case changing the system regional settings after installation using the command-line can be very useful and time-saving

As outlined in the [Vista International Settings Guide](https://msdn.microsoft.com/en-us/goglobal/bb964650(en-us).aspx), you must create an XML File which would be then executed by the regional settings control panel applet.

# Creating the XML file

There are plenty of options, so here is an example which allows to change the locale to English (United Kingdom) for all users (system and new users)

```
<pre class="lang:xhtml decode:true " title="Changing user locale to ENglish UK for All users"><gs:GlobalizationServices xmlns:gs="urn:longhornGlobalizationUnattend">
 <gs:UserList>
 <gs:User UserID="Current" CopySettingsToDefaultUserAcct="true" CopySettingsToSystemAcct="true"/>
 </gs:UserList>
 <!--User Locale-->
 <gs:UserLocale> 
 <gs:Locale Name="en-GB" SetAsCurrent="true"/> 
 </gs:UserLocale>
 <!-- Non Unicode programs -->
<gs:SystemLocale Name="en-GB" />
</gs:GlobalizationServices>
```

There are 3 groups to outline here:

- the &lt;gs:UserList&gt; tag tells which users will be impacted. Here the 2 attributes CopySettingsToDefaultUserAcct and CopySettingsToSystemAcct imply the setting will be copied into HKEY\_USERS\\.Default and the System profile.
- the &lt;gs:UserLocale&gt; define the locales and keyboard we’ll be adding. Here we just change the locale
- the &lt;gs:SystemLocale&gt; is used for all non-modern programs which do not use specific settings.

Once you’ve create this XML file, you must pass it to the following command:

```
<pre class="lang:default decode:true " title="Passing a User Locale XML Settings">control intl.cpl,, /f:"c:\path\newlocale.xml"
```

# Checking results in the logs

If you need to check if the settings have been properly applied, you must look into the Applications and Services Logs, Microsoft, Windows, International in the Operational Logs. A successful change is event 13000 under Task Category “NLS Unattended and Regional Settings Options”[![](https://dimitri.janczak.net/wp-content/uploads/2017/08/Windows-SetLocale-NLSOperations-EventID-13000.png)](https://dimitri.janczak.net/wp-content/uploads/2017/08/Windows-SetLocale-NLSOperations-EventID-13000.png)

If you specified CopySettingsToDefaultUserAcct and everything is valid, Event ID 13009 will be present:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/08/Windows-SetLocale-NLSOperations-EventID-13009-DefaultUser.png)](https://dimitri.janczak.net/wp-content/uploads/2017/08/Windows-SetLocale-NLSOperations-EventID-13009-DefaultUser.png)

If you specified CopySettingsToSystemAcct and everything is valid, Event ID 13010 will be present:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/08/Windows-SetLocale-NLSOperations-EventID-13010-SystemAccount.png)](https://dimitri.janczak.net/wp-content/uploads/2017/08/Windows-SetLocale-NLSOperations-EventID-13010-SystemAccount.png)

The list of [Windows available locales](https://msdn.microsoft.com/en-us/library/cc233982.aspx) will guide you the list of locales you can use. Please be sure to read this list as abbreviations may differ from common sense (e.g. English (UK) is en-GB, not en-UK).