---
id: 952
title: 'Trial period reset of Visual Studio Community Edition'
date: '2019-07-13T18:26:07+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=952'
permalink: /2019/07/13/trial-period-reset-of-visual-studio-community-edition/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:953;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2019/07/visual_studio_community_2017_web_page.jpg
categories:
    - troubleshooting
    - 'Visual Studio'
tags:
    - 'powershell module'
    - troubleshooting
    - 'Visual Studio'
---

Although Visual Studio Community Edition is supposed to be free, you are required after 30 days by the graphical interface to have an account on Microsoft’s online services to continue to use the product. The trial period reset of Visual Studio Community Edition can however be easily achieved by editing the registry.

As mentioned in this [StackOverflow question](https://stackoverflow.com/questions/43390466/is-visual-studio-community-a-30-day-trial/51570570#51570570), the expiration date for these 30 days are registered in a registry value protected by the CryptProtectData API. The algorithm is then

1. Get the value in HKCR\\Licenses\\5C505A59-E312-4B89-9508-E162F8150517\\08878 for VSCE 2017 or HKCR\\Licenses\\41717607-F34E-432C-A138-A3CFD7E25CDA\\09278 for VCSE 2019
2. Decrypt the value with `<a href="https://docs.microsoft.com/en-us/windows/win32/api/dpapi/nf-dpapi-cryptunprotectdata">CryptUnprotectData</a>`.
3. Get and change the values in bytes -16,-15 (YY), -14,-13 (MM), -12, -11 (DD)
4. Encrypt the value with `<a href="https://docs.microsoft.com/en-us/windows/win32/api/dpapi/nf-dpapi-cryptprotectdata">CryptProtectData</a>`.
5. Set the value back in the registry where you found it.

In Powershell, a module has been published which does this. However, the manifest was not properly edited, so you can find an [updated version](https://github.com/1Dimitri/VSCELicense/) with the following changes:

1. CompatiblePSEditions is valid for Powershell 5.1 or higher
2. The RequiredAssemblies to load, in this case System.Security for the Cryptography functions, directive takes an array of strings, not a simple string

If you want a ready-to-use solution, i.e. a MSI Installer which installs the module without the need for an internet connection and a scheduled task to periodically reset the period, just go to the[github repo for this project](https://github.com/1Dimitri/VSCELicense/tags).