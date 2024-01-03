---
id: 450
title: 'UniCERT PKI: limited auto-enrollment support'
date: '2016-07-07T21:21:56+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=450'
permalink: /2016/07/07/unicert-pki-limited-autoenrollment-support/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"453";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/07/PKI-Certificate.gif
categories:
    - PKI
tags:
    - auto-enrollment
    - Certificates
    - CyberTrust
    - firewall
    - PKI
    - UniCERT
    - Verizon
---

In enterprise environments, non-Windows PKI solutions are not uncommon. Such as product is [PKI from Verizon CyberTrust called UniCERT](http://www.pkiservices.intesa.it/verizon_pki.html), or UniCERT PKI for short.

Although the product delivers standard PKI features, like many Unix-Java based products it has many limitations when it comes to integration into the Windows world. Here is a few caveats I came across when I had to participate into such an integration.

## Use as a stand alone CA

If you use it as regular stand-alone Certification Authority, standard certificates are not a big deal. Some limitations however appear because what they call ‘policies’ do not target Windows specifically. In UniCERT jargon, a policy is roughly what a Microsoft CA calls a template. Since most of the Microsoft template equivalents are not provided by UniCERT, everw time you design a policy for a Windows machine or user, you have to create a policy by looking at the Microsoft template and creating all the needed OIDs and policies. This is just fastidious.

## Use as an Enterprise CA

However the real culprit occurs when you want to create Enterprise CAs. An Enterprise CA is supposed to publish some records into the Active Directory so the Windows Client can find it and automatically request and receive certificates if needed.

As the product is Unix and Java based, they had to mimic the functionality, including the Windows protocol and the records in the AD. So they create an Auto-Enrollment module you have to install on a member server. And here are the problems:

1. They don’t give it for free : you have to pay for this module in addition to the Windows license for the member server if any;
2. The module is not HA/DRP compliant: you cannot install several member servers having each the module in an active/active or even active/passive fashion. Needless to say that the failover clustering configuration has not been tested
3. The module doesn’t install correctly if you don’t grant the account you’re uninstalling under rights to the configuration AD Container: it should be able to read and write on this container and all the objects below
4. The module doesn’t create CA templates for the Windows machines and users to consume. You have to install AD Certificate Services and then uninstall it. So that the templates are published by Microsoft product. Of course, none of the document tells you about doing this with a clean CAPolicy.Inf file. Therefore you end up with all the templates created in your Active Directory

## Additional security issues

1. The code running on the member server which acts a as relay to your UniCERT PKI infrastructure is based on Java, so you’ll get all the JAVA vulnerabilities and patch cycle to follow for free, without Verizon telling you if you can safely update JAVA or not
2. Finally, the code still implements a pre-Windows 2008R2 PKI; therefore you must deploy and pay for one server module per forest even if you have forest trust relationships
3. A side-effect of being stuck in a pre-Windows2008R2 protocol version is that you cannot propose: 
    - autoenrollment over HTTPS or to non-domain-joined machines
    - your firewall has to open all the set of ports for RPC: TCP/135 and the dynamic ports