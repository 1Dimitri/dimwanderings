---
id: 346
title: 'Registering a custom ADFS MFA provider the easy way'
date: '2015-08-14T19:29:21+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=346'
permalink: /2015/08/14/registering-a-custom-adfs-provider-the-easy-way/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"354";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/08/MultiForm-Authentication-ADFS-PIN-CODE-example.png
categories:
    - 'ADFS-AD Federation Services'
tags:
    - Assembly
    - GAC
    - MFA
    - 'Multi-Form Authentication'
    - Register-ADFSProvider
---

Starting with Windows Server 2012 R2, and as [demonstrated there](http://blogs.technet.com/b/cloudpfe/archive/2014/02/01/how-to-create-a-custom-authentication-provider-for-active-directory-federation-services-3-0-part-2.aspx), you can create yourself your own multi-form factor authentication in any .NET language that can create interfaces. Once you have created your DLL assembly, you must register it with [Register-AdfsAuthenticationProvider](https://technet.microsoft.com/en-us/library/dn479353.aspx). However the syntax for the TypeName argument is somewhat cumbersome. In the post already mentioned, many steps are necessary; however you can automate many of these.

In particular there is no need for the sn.exe tool to be run as the public token can be retrieved once the assembly is registered.

Therefore, a simplified procedure is:

1. Copy the DLL into some directory
2. Register the DLL Assembly in the GAC. If you do not know do how to do it, refer to [this post](http://dimitri.janczak.net/2015/08/14/registering-a-dll-assembly-in-the-gac-using-powershell/).
3. Create the Typename parameter contents by gluing the following two parts: 
    - the namespace qualified name of the class implementing the IAuthenticationProvider. If you do not this information look at the different types the assembly implement and throw away types with Metadata or Presentation in their names
    - a comma and a space
    - the DLL assembly full name
4. Run the cmdlet

Here is a step-by-step example

```
<pre class="lang:ps decode:true " title="Register-ADFSProvider provider">$d = ([system.reflection.assembly]::loadfile("C:\Program Files\OEM\ADFS\ADFSMFASimplePINCode.dll"))
$df = $d.FullName
# $df now contains the second part of what we'll need
#eg.
# ADFSMFASimplePINCode, Version=1.0.0.0, Culture=neutral, PublicKeyToken=deadbeef5749c6f3

$dt = $d.DefinedTypes
# $dt is a table whose one of the entry will contain the first part
# IsPublic IsSerial Name                                     BaseType
# -------- -------- ----                                     --------
# False    False    SimplePINAdapterPresentation             System.Object
# False    False    SimplePINAuthenticationMetadata          System.Object
# False    False    SimplePINAuthenticationProvider          System.Object
$interface = $dt | ? { $_.ImplementedInterfaces.Name -eq 'IAuthenticationAdapter'  }
$fullclassname =  $interface | Select FullName
$shortclassname =  $interface | Select Name

# fullclassname will contain the first part of the expression such as
#ADFSMFASimplePINCode.SimplePINAuthenticationProvider
$regstring = $classname +', '+$df
# Now register:
 Register-AdfsAuthenticationProvider -TypeName $regstring -Name $shortclassname
# The warning is OK, just restart ADFSSvc
# WARNING: PS0114: The authentication provider was successfully registered with the policy store.
# To enable this provider, you must restart the AD FS Windows Service on each server in the farm.


```

Do not forget that the DLL must be deployed on all the AD Federation Services servers of your farm, but there’s no need to deploy it on the AD FS proxies.

If you are interested in, the code for the dll mentioned above is available at [github](https://github.com/1Dimitri/SimpleMFAPinCode).