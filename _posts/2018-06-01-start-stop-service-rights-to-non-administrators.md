---
id: 876
title: 'Start Stop service rights to non administrators'
date: '2018-06-01T14:44:06+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=876'
permalink: /2018/06/01/start-stop-service-rights-to-non-administrators/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:873;s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2018/06/wuauserv-service-registry-security.png
categories:
    - 'Windows OS'
tags:
    - administrator
    - 'least privilege'
    - SDDL
    - security
    - service
---

LIke any other object in the Windows world, services are objects, you can therefore give users rights on them. Let’s see for example how to allow start stop service rights to non-administrators.

In order to change the permissions to a service, you’ll need an account who has rights to change permissions on that service obviously. This means being local administrator of the box or already having that “Full control” right on the service.  
You’ll also need to retrieve the SID of the users or groups you want grant access.  
There are a few ways to achieve this. You could use a runas command to launch a “whoami /all” command for example. For a domain user, you can use a simple “Get-ADUser” / “Get-ADGroup” cmdlet

You also need the short name of the service. In the full example below, I’ll take “Windows Update” service. To retrieve its short name, I can use either

```
<pre class="lang:batch decode:true " title="sc to get service name">sc getkeyname "Windows Update"
[SC] GetServiceKeyName SUCCESS
Name = wuauserv
```

or in Powershell

```
<pre class="lang:ps decode:true" title="Powershell long short service names ">Get-Service | ? { $_.DisplayName -eq 'Windows Update'}

Status Name DisplayName
------ ---- -----------
Running wuauserv Windows Update
```

Once we get that short name also called «keyname » (Because it is the name of the key in the registry under which the parameters for the service are stored), we can dump its current security settings. Since we can easily mess up, we’ll save the current settings for future retrieval. In order to accomplish this, you do:

```
<pre class="lang:default decode:true " title="security descriptor of a service using sc">sc sdshow wuauserv
```

And you’ll get something as ugly as:

```
<pre class="lang:batch decode:true " title="SDDL default seurity decriptor for a service">D:(A;;CCLCSWRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)S:(AU;FA;CCDCLCSWRPWPDTLOSDRCWDWO;;;WD)
```

This is a security descriptor in [SDDL format](https://msdn.microsoft.com/en-us/library/windows/desktop/aa379570(v=vs.85).aspx) which is stored in the Security value of the Security registry key under the keyname in HKLM\\CurrentControlSet\\Services.

[![](https://dimitri.janczak.net/wp-content/uploads/2018/06/wuauserv-service-registry-security.png)](https://dimitri.janczak.net/wp-content/uploads/2018/06/wuauserv-service-registry-security.png)

Let’s split in parts:

```
<pre class="lang:default decode:true " title="SSDL Overall structure">D:<Something>S:<SomethingElse>
```

The &lt;Something&gt; after the D: is what we are interested in, these are the security permissions.  
The &lt;SomethingElse&gt; after the S: are the audit permissions, we are not interested in here. Notice how letters are counter intuitive  
After the D:, you’ve got a set of blocks between parenthesis. Each represent a security right and must be read as follows:  
The first letter is A for Allow or D for Deny (you remember we can deny people, don’t you?)  
The second part after the 2 semicolons is the permission set and the letters depend on the object you’re working on. You must read them by pair.  
For a service, you’ve got:

- CC – SERVICE\_QUERY\_CONFIG – read the configuration of the service
- LC – SERVICE\_QUERY\_STATUS – read the status of the service from the Service Control Manager
- SW – SERVICE\_ENUMERATE\_DEPENDENTS – list dependencies
- LO – SERVICE\_INTERROGATE – ask the service its current status
- CR – SERVICE\_USER\_DEFINED\_CONTROL – send a service control command which only make sense service by service (like the custom attributes for an Exchange object for example)
- RC – READ\_CONTROL – read the security permissions
- RP – SERVICE\_START – start the service
- WP – SERVICE\_STOP – stop the service
- DT – SERVICE\_PAUSE\_CONTINUE – pause / continue the service

The last part, just before the end parenthesis is the user or group the permission applies to. As we have the default set of permissions here, we only have “well-known“ groups which may be abbreviated by 2 letters (Otherwise you would have for each domain to know the full SID even the RID wouldn’t change)  
Often you’ll encounter

- SY – System
- BA – Administrators (think “Built-in administrators”)
- DA – Domain administrators
- DU – Domain Users
- AU – Authenticated Users
- WD – Everyone (sounds like ‘World’, unix roots of the NT native subsystem)

So we can now build the security descriptor to add to grant the start and stop rights we want:

- my user SID is S-1-5-21-1111111111-4444444444-2000000000-1158
- Grant means A
- Start and Stop are RP and WP respectively
- Therefore our part of the SDDL will be: (A;;RPWP;;S-1-5-21-1111111111-4444444444-2000000000-1158)

Since we don’t want to remove permissions for other users we must reassemble the full string:

```
<pre class="lang:default decode:true " title="SDDL to grant start stop service right to any user">D:(A;;CCLCSWRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)(A;;RPWP;;S-1-5-21-1111111111-4444444444-2000000000-1158)S:(AU;FA;CCDCLCSWRPWPDTLOSDRCWDWO;;;WD)
```

Do not forget we must put this in the “D:” part of the SDDL!  
Once we’ve built that string, we can use

```
<pre class="lang:batch decode:true " title="sc command to set security descriptor on a service">sc sdset wuauserv D:(A;;CCLCSWRPLORC;;;AU)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)(A;;RPWP;;S-1-5-21-1111111111-4444444444-2000000000-1158)S:(AU;FA;CCDCLCSWRPWPDTLOSDRCWDWO;;;WD)
```

The return should be

```
<pre class="lang:batch decode:true " title="sc set security descriptor success">[SC] SetServiceObjectSecurity SUCCESS
```

You may get errors such as

```
<pre class="lang:default decode:true " title="sc set security descriptor error">No mapping between account names and security IDs was done.
```

In that case, check you have a valid SID and there is no space in all the string.

You can now check that your regular user can issue command like ‘sc start wuauserv’ or ‘sc stop wuauserv’

You can also verify that the registry value “Security” under HKLM\\CurrentControlSet\\Services\\wuauserv\\Security had its contents changed.  
Once you’ve made the change on a server, you can replicate the setting to other machines using your preferred method (script, GPP, …)