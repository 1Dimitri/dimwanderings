---
id: 363
title: 'ADFS 3.0 Firewall Ports in root-child domains'
date: '2015-08-26T22:14:03+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=363'
permalink: /2015/08/26/adfs-3-0-firewall-ports-in-root-child-domains/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"364";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/08/ADFS-architecture-Root-Child-Domain.png
categories:
    - 'ADFS-AD Federation Services'
tags:
    - child
    - firewall
    - ldap
    - root
---

There is a lot of documentation about AD FS 3.0, the Active Directory Federation Services that comes with Windows 2012 R2. For example, this [series ](http://blogs.technet.com/b/askpfeplat/archive/2013/12/09/how-to-build-your-adfs-lab-on-server-2012-part-1.aspx)of tutorials walks you through the different steps to build a lab.

However almost all articles assume that you have a simple single domain forest to work with. When your forest has a root domain and one or more child domains, there are a few things which are not very well documented:

- If you don’t have gMSA account, the account to run the AD FS service can reside in the child domain you’re federating. It doesn’t need additional rights in the root domain
- The AD FS Servers (not the proxies, but the internal member servers) need to talk to the Domain Controllers of the domain you’re federating, which may seem obvious. However they also need to speak to the Domain Controllers of the root domain. If you have firewall between the AD FS Server and those Domain Controllers you must open the standard ports for the federation to work. Symptoms include error messages such as Event ID 364 ```
    <pre class="prettyprint prettyprinted">Encountered error during federation passive request.
    ...
    Microsoft.IdentityServer.Protocols.Saml.NoAuthenticationContextException: MSIS7012:
    An error occurred while processing the request. Contact your administrator for details.
    
    System.ComponentModel.Win32Exception (0x80004005): Failed to open ldap conection to yourdomain.com
    ```
    
    Opening the LDAP, LDAPS or GC ports is not enough, the regular range between a member server and a Domain Controller, as shown on the following picture:[![ADFS-architecture-Root-Child-Domain](http://dimitri.janczak.net/wp-content/uploads/2015/08/ADFS-architecture-Root-Child-Domain.png)](http://dimitri.janczak.net/wp-content/uploads/2015/08/ADFS-architecture-Root-Child-Domain.png)