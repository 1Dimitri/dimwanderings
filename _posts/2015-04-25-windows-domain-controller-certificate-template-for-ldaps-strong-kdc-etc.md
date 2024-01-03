---
id: 126
title: 'Windows Domain Controller Certificate template for LDAPS, Strong KDC, etc.'
date: '2015-04-25T15:32:44+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=126'
permalink: /2015/04/25/windows-domain-controller-certificate-template-for-ldaps-strong-kdc-etc/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:147;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/04/certificate-generic.png
categories:
    - 'Windows OS'
tags:
    - 'Active Directory'
    - authentication
    - Certificates
    - 'domain controller'
    - LDAPS
    - PKI
---

To perform LDAPS with Domain Controllers, you must install a certificate into the personal store of the computer account.  
If you are using Windows Enterprise CAs, it is no problem, as a dedicated template used to exist for a while.  
For 3rd-party CAs, until Windows 2003, the requirements the certificate must fulfill were outlined in [KB 321051](http://support.microsoft.com/kb/321051/en-us).  
The requirements for WIndows 2008 R2 domain controllers were listed in [that blog entry](http://social.technet.microsoft.com/wiki/contents/articles/3824.updated-requirements-for-a-windows-server-2008-r2-domain-controller-certificate-from-a-3rd-party-ca.aspx).

However, if you are not familiar with PKI and/or PKI tools you may find this article a bit hard to read. Here is the step-by-step procedure:

1. create a mydc-req.inf with the contents attached to this post on the Domain Controller you want to have a certificate for
2. issue a certreq -new mydc-req.inf mydc-req.req <span class="anchor" id="line-43"></span>
3. send the mydc-req.req for signing <span class="anchor" id="line-44"></span>
4. save the answer as mydc.crt (you mentioned you wanted a PKCS#10) <span class="anchor" id="line-45"></span>
5. Do not forget to add any public key of any CA from the signing chain into the 3rd party CA store of the local computer
6. issue a certreq -accept mydc.crt <span class="anchor" id="line-46"></span>

Some notes:

- If you created the request with certreq, you must accept it by using certreq; if you use another tool, use that tool to finish the certification process (e.g. mmc snap-in) <span class="anchor" id="line-48"></span>
- This is a **certificate** known as KDC authentication, it deviates from the regular LDAPS Win2003, but allows more <span class="anchor" id="line-49"></span>
    - LDAPS (that’s the subject part) <span class="anchor" id="line-50"></span>
    - KDC signing with reference to the domain from the calling client, not a particular Domain Controllrer (that’s the SAN -Subject Alternate Name- part) <span class="anchor" id="line-51"></span>
    - use of smart card hosted **certificate** for the clients to access the LDAPS/KDC support

```
<pre class="lang:ini decode:true " title="Domain Controller Certificate template">[Version] 
Signature="$Windows NT$"

[NewRequest]
Subject = "CN=mydc.mydomain.myforest.dom" ; replace with the FQDN of the DC for LDAPS
KeySpec = 1 
KeyLength =  2048
; Can be 1024, 2048, 4096, 8192, or 16384. 
; Larger key sizes are more secure, but have ; a greater impact on performance. 
; 2048 = ANSSI recommendation until 2020
Exportable = TRUE 
MachineKeySet = TRUE 
SMIME = False 
PrivateKeyArchive = FALSE 
UserProtected = FALSE 
UseExistingKeySet = FALSE 
ProviderName = "Microsoft RSA SChannel Cryptographic Provider" 
ProviderType = 12
RequestType = PKCS10 
KeyUsage = 0xa0 

[EnhancedKeyUsageExtension] 
OID=1.3.6.1.5.5.7.3.1 ; this is for Server Authentication
OID=1.3.6.1.5.5.7.3.2 ; this is for Client Authentication
OID=1.3.6.1.5.2.3.5 ; this is for KDC Authentication
OID=1.3.6.1.4.1.311.20.2.2 ; this is for Smart Card Logon

; For Windows 2008 and higher only
[Extensions]
<span class="anchor" id="line-31"></span>2.5.29.17 = "{text}"
<span class="anchor" id="line-32"></span>_continue_ = "dns=mydomain&"
<span class="anchor" id="line-33"></span>_continue_ = "dns=mydomain.myforest.dom&" 
; For Windows 2003 only ---
[RequestAttributes]
SAN="dns=mydomain.myforest.dom"
SAN="dns=mydomain"  ; NetBIOS Domain Name
```