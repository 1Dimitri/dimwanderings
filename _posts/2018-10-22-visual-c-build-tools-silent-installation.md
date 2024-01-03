---
id: 891
title: 'Visual C++ Build Tools silent installation'
date: '2018-10-22T15:26:44+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=891'
permalink: /2018/10/22/visual-c-build-tools-silent-installation/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:894;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/10/Visual-Studio-Installer.png
categories:
    - 'Visual Studio'
tags:
    - c++
    - developer
    - msbuild
    - sdk
    - 'Visual Studio'
---

With the release of Visual Studio 2017, the creation of offline distribution files in order to achieve the Visual C++ build tools silent installation has become easier than in previous versions.

The key parameter is the –layout switch which allows to specify where you want to store the offline files.

The –lang switch is very useful as some options will download localized versions for several languages you may have no need for, e.g. the various .NET framework redistributables.

The –add can then be used to specify the components you want to store locally for future installations.

To compile C++ programs without too much hassle, as written down in the [Visual C++ and build tools presentation article](https://blogs.msdn.microsoft.com/vcblog/2016/11/16/introducing-the-visual-studio-build-tools/), you should use Microsoft.VisualStudio.Workload.VCTools keyword to install those tools.

```
<pre class="lang:batch decode:true " title="Offline creation of Visual C++ Build Tools">rem Target Directory = C:\VSBuildTools17
rem English only
rem Include all recommended and optional tools
vs_buildtools.exe --layout C:\VSBuildTools17 --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended --includeOptional --lang en-US
```

Additionally, you can include recommended and optional software. In our case VC++ recommended and optional tools are as follows:

[![](https://dimitri.janczak.net/wp-content/uploads/2018/10/Visual-Studio-Installer_VSBuildTools.png)](https://dimitri.janczak.net/wp-content/uploads/2018/10/Visual-Studio-Installer_VSBuildTools.png)

- –includeOptional notably gives you the Windows 10 SDK, the CMake which are very useful to compile SDK samples.

Please note that the vs\_build\_tools.exe you can find on[ that download page](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=15) will still result in creating a vs\_setup.exe in the target directory.

You can also add –quiet so no window is displayed.

You can find the [full list of command line parameters](https://docs.microsoft.com/en-us/visualstudio/install/use-command-line-parameters-to-install-visual-studio?view=vs-2017) on Microsoft’s site.