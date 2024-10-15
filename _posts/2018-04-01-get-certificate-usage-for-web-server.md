---
id: 847
title: 'Get-Certificate usage for Web Server'
date: '2018-04-01T10:13:51+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=847'
permalink: /2018/04/01/get-certificate-usage-for-web-server/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:848;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/04/web-server-certificate-template-properties.jpg
categories:
    - 'ADCS-Certificate Services'
tags:
    - Certificate
    - 'certificate template'
    - get-certificate
    - PKI
    - powershell
---

The documentation for the powershell cmdlet [Get-Certificate](https://docs.microsoft.com/en-us/powershell/module/pkiclient/get-certificate?view=win10-ps) only use generic examples. In this post, let’s see the Get-Certificate usage for Web Server.

In our scenario, you have an Enterprise CA whose service is published under the name ‘My Company SubCA I’.

You also have [duplicated the Web Server template](https://kb.vmware.com/s/article/2062108) under the name ‘WebServerCertificate10’. Please note that this template name doesn’t exist by default and is only a decision of mine. You have issued that template, and granted permissions to enroll this template to the computer ‘wsus01’ (or put that computer in a group and assigned permissions to that group)

We’d like to issue a certificate for a web server under an alias such as ‘wsus.mycompany.com’ whereas the server is named wsus01.mycompany.com

Let’s review the arguments you will need

- URL is in fact the URI of the Enterprise CA endpoint, meaning that this setting may also be linked to the LDAP Distinguished name. Here we will use LDAP:///CN=My Company SubCA I. Please note that the spaces within the name will make the use of double quotes mandatory.
- DNSName are the name under the server will be resolved by the client. You may not publish the real name of the server but the alias is mandatory
- The CertificateStoreLocation is where you want the new certificate to be put. If you don’t specify the location chances are the certificate will be put in the personal store of the current user; that’s definitively not what we want here, we’ll put it in the personal store of the local machine
- SubjectName is the name of the web site using the CN= notation.

Our command will then be:

```
Get-Certificate -URL "ldap:///CN=My Company SubCA I" `
-SubjectName "wsus.mycompany.com" `
-DnsName wsus.mycompany.com,wsus01.company.com `
-Template WebServerCertificate10 `
-CertificateStoreLocation Cert:\LocalMachine\My
```

If everything goes OK, you’ll get a line stating the certificate was issue and you’ll get its thumbprint.

If you get an error ASN.1, double check you didn’t have omitted any CN= prefix in the above command.