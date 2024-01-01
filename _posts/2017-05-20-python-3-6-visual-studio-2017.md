---
id: 723
title: 'Python 3.6 and Visual Studio 2017'
date: '2017-05-20T17:22:04+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=723'
permalink: /2017/05/20/python-3-6-visual-studio-2017/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"728";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/05/Visual-Studio-2017-Python.png
categories:
    - 'Python Build and package'
tags:
    - 'Build Tools'
    - pip
    - setuptools
    - vcvarsall
    - 'Visual Studio'
---

As a new version of Visual Studio comes out, there are often fixes to put in your build chain tools so packages with external C/C++ compilation requirements still work. For Python 3.6 and Visual Studio 2017 to work together, a good reference start point is the [Windows compilers compatibility list on python.org](https://wiki.python.org/moin/WindowsCompilers).

Let’s try to compile scrapy which has a lot of dependencies including the twisted package.

Let’s assume you have installed “Visual Studio 2017 Community” on your PC with the C++ support. Trying to run the well-known

```
<pre class="lang:default decode:true " title="scrapy install command">pip install scrapy
```

will end-up with a compile error, such as the following:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/05/SCrapy-Twisted-Build-Error.png)](https://dimitri.janczak.net/wp-content/uploads/2017/05/SCrapy-Twisted-Build-Error.png)

```
<pre class="lang:batch decode:true" title="Error message building twisted">"   building 'twisted.test.raiser' extension
    error: Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools": http://landinghub.visualstudio.com/visual-cpp-build-tools
    ----------------------------------------
Command ""c:\program files\python36\python.exe" -u -c "import setuptools, tokenize;__file__='C:\\Users\\ADMINI~1\\AppData\\Local\\Temp\\pip-build-68oh6ofn\\Twisted\\setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record
 C:\Users\ADMINI~1\AppData\Local\Temp\pip-paytsys9-record\install-record.txt --single-version-externally-managed --compile" failed with error code 1 in C:\Users
\ADMINI~1\AppData\Local\Temp\pip-build-68oh6ofn\Twisted\"
```

When working with Python 3.6 you may find however [many stackoverflow articles](http://stackoverflow.com/questions/29846087/microsoft-visual-c-14-0-is-required-unable-to-find-vcvarsall-bat) stating that you must install Visual Studio 2015 back. Save yourself time and space by not doing it.

As mentioned as a requirement in the previously mentioned article, one requirement is to have a version of the setuptools library which is higher than 34.0. Unfortunately, Python 3.6 comes with 28.0 and the simple

```
<pre class="lang:default decode:true" title="simple pip install command">pip install setuptools
```

won’t work as it tells you you have already what’s needed. Use instead:

```
<pre class="lang:batch decode:true " title="forcing upgrade of setuptools package">python -m pip install -U pip setuptools
```

If everything goes well, you’ll have:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/05/pip-install-upgrade-setup-tools.png)](https://dimitri.janczak.net/wp-content/uploads/2017/05/pip-install-upgrade-setup-tools.png)

This will allow Python to access your Visual Studio 2017 installation and no longer have two versions or more installed in parallel.

The last trick is to use the proper environment to run command-line installations. With Visual Studio 2017 come multiple command prompts setups:

[![](https://dimitri.janczak.net/wp-content/uploads/2017/05/Visual-Studio-2017-Command-Line-Prompt-Icons.png)](https://dimitri.janczak.net/wp-content/uploads/2017/05/Visual-Studio-2017-Command-Line-Prompt-Icons.png)

Do not choose ‘Developer’ although it may sound appealing. You’ll get caught with LNK2010 linking errors because of INCLUDE issues, in the same way that a regular cmd would output the following error:

```
<pre class="lang:default decode:true" title="Error when compiling with regular cmd">    C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.10.25017\bin\HostX64\x64\cl.exe /c /nologo /Ox /W3 /GL /DNDEBUG /MD -DWIN32=1
"-Ic:\program files\python36\include" "-Ic:\program files\python36\include" "-IC:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.1
0.25017\Include" "-IC:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.10.25017\ATLMFC\Include" "-IC:\Program Files (x86)\Windows Ki
ts\10\include\10.0.15063.0\ucrt" "-IC:\Program Files (x86)\Windows Kits\NETFXSDK\4.6.1\include\um" /Tcsrc/twisted/test/raiser.c /Fobuild\temp.win-amd64-3.6\Rele
ase\src/twisted/test/raiser.obj
    raiser.c
    c:\program files\python36\include\pyconfig.h(222): fatal error C1083: Cannot open include file: 'basetsd.h': No such file or directory
    error: command 'C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\VC\\Tools\\MSVC\\14.10.25017\\bin\\HostX64\\x64\\cl.exe' failed with exit
 status 2

Command ""c:\program files\python36\python.exe" -u -c "import setuptools, tokenize;__file__='C:\\Users\\ADMINI~1\\AppData\\Local\\Temp\\pip-build-amp2cfl7\\Twisted\\setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record
C:\Users\ADMINI~1\AppData\Local\Temp\pip-5lwfhlfl-record\install-record.txt --single-version-externally-managed --compile" failed with error code 1 in C:\Users
\ADMINI~1\AppData\Local\Temp\pip-build-amp2cfl7\Twisted\
```

For proper compilation, choose the native architecture. In my case, I Choose ‘x64 Native Tools’ and finally got the following successful result:[![](https://dimitri.janczak.net/wp-content/uploads/2017/05/scrapy-build_lib.win-amd64-3.6_twisted_test_raiser.cp36-win_amd64.pyd_.png)](https://dimitri.janczak.net/wp-content/uploads/2017/05/scrapy-build_lib.win-amd64-3.6_twisted_test_raiser.cp36-win_amd64.pyd_.png)