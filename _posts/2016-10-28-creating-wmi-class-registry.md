---
id: 521
title: 'Creating a WMI Class from registry entries'
date: '2016-10-28T17:11:03+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=521'
permalink: /2016/10/28/creating-wmi-class-registry/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"522";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/10/WMI-infrastructure.jpg
categories:
    - code
tags:
    - inventory
    - mof
    - powershell
    - registry
    - template
    - wmi
---

As you may know, [Windows Management Instrumentation](https://msdn.microsoft.com/en-us/library/aa394582(v=vs.85).aspx) (WMI) is the way Microsoft has made available many API available since Windows XP over the time. Creating a WMI class can be a tedious class but creating a WMI class from registry entries is in contrast fairly easy.

# Creating one’s own WMI Class Steps

As you perhaps know, [creating WMI](https://www.microsoft.com/en-us/download/details.aspx?id=8572) objects is a matter of

- describing the objects and its instances, usually in a MOF format
- Compiling the description into something useable by the WMI engine

# Registry use in WMI

Registry has its own provider in the WMI world in order to easily create or update which allows a registry key and its values to be mapped to a WMI class.

In order for you to use the registry in a WMI, you must then

1. register the WMI provider in the MOF File
2. create a new class
3. for each property of this class, indicate which registry key stores the underlying data.

The only problem is that every WMI class instance must have a unique identifier. Fortunately the plumbing inside the registry provider allows us to define a dummy property called InstanceKey that will do the trick once it has been prefixed by the \[key\] keyword.

# WMI Class template

The following template uses the [eps engine](https://github.com/straightdave/eps) which brings Razor equivalent templating capabilities to Powershell:

```
<pre class="lang:c# decode:true" title="WMI Class creation from MOF File ">// create WMI Class that maps a registry key
// DJ - 1.0 - 18.10.2016
//==================================================================
// 1. Register Registry property provider (shipped with WMI)
//==================================================================

#pragma namespace("\\\\.\\root\\cimv2")

// Registry instance provider
instance of __Win32Provider as $InstProv
{
	Name    ="RegProv" ;
	ClsID   = "{fe9af5c0-d3b6-11ce-a5b6-00aa00680c3f}" ;
	ImpersonationLevel = 1;
	PerUserInitialization = "False";
};

instance of __InstanceProviderRegistration
{
	Provider    = $InstProv;
	SupportsPut = True;
	SupportsGet = True;
	SupportsDelete = False;
	SupportsEnumeration = True;
};


// Registry property provider
instance of __Win32Provider as $PropProv
{
	Name    ="RegPropProv" ;
	ClsID   = "{72967901-68EC-11d0-B729-00AA0062CBB7}";
	ImpersonationLevel = 1;
	PerUserInitialization = "False";
};

instance of __PropertyProviderRegistration
{
	Provider     = $PropProv;
	SupportsPut  = True;
	SupportsGet  = True;
};


//==================================================================
// 2. <%= $Classname %>
//==================================================================

#pragma namespace ("\\\\.\\root\\cimv2")

// Class definition

#pragma deleteclass("<%=$Classname%>",nofail)
[DYNPROPS]
class <%= $classname %>
{
// unique identifier for the class
	[key]
	string InstanceKey;
// 3.
<% $properties | % { %>
   string <%= $_ %>;
<% } %>
};


// Instance definition

[DYNPROPS]
instance of <%= $Classname%>
{
	InstanceKey = "@";

<% $properties | % { %>
   [PropertyContext("local|<%=$keyexpanded%>|<%=$_%>"), Dynamic, Provider("RegPropProv")]
   <%= $_ %>;
<% } %>

};
```

You can then use the above file saved as ‘WMItemplate.txt’ in a command like:

```
<pre class="lang:ps decode:true">$moftext = Expand-Template -file 'WMITemplate.txt' -safe -binding @{Key='HKLM:\Software\MySoftware',@keyexpanded='HKEY_LOCAL_MACHINE\\Software\\MySoftware,@classname='MyCompany_MySoftware'}
$moftext | Set-Content 'mycompany_mysoftware.mof' -force
mofcomp -autorecover -f mycompany_mysoftware.mof
```

KeyExpanded is just the not abbreviated version of describing a registry key, where you do not use HKLM but HKEY\_LOCAL\_MACHINE and replace simple backslashes by double backslashes