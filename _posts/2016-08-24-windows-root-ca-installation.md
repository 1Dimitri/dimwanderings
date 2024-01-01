---
id: 494
title: 'Windows Root CA Installation'
date: '2016-08-24T19:40:25+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=494'
permalink: /2016/08/24/windows-root-ca-installation/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"500";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/08/Windows-PKI-Hierarchy.gif
categories:
    - 'ADCS-Certificate Services'
    - PKI
tags:
    - 'best practices'
    - CA
    - Certificates
    - installation
    - powershell
    - tutorial
---

As opposed to other roles installation, a Windows Root CA installation is not a simple question of clicking next and finish buttons.  
Since the root CA will be offline most of the time, you can use a virtual machine. HOwever, please note that your hypervisor administrators must be considered as PKI administrators if you don not take additional security measures (encryption, audit, …)

## Preparing the machine

Machine Specifications are not demanding :

- 2 GB RAM.
- 40 GB C drive or more if you want to take care of patching the system regularly.
- Network Card, which can be disabled after the crl and crt are transferred

It is so important to secure the Root CA server. Here is a [blue print ](https://technet.microsoft.com/en-us/library/dn786426.aspx) to be tailored to your needs.

```
<pre class="lang:default decode:true" title="CA Feature Installation">Install-WindowsFeature Adcs-cert-Authority -IncludeManagementTools
Install-WindowsFeature RSAT-ADCS-Mgmt
```

## Installing the role: don’t do click, click, next

The CAPolicy.inf file provides Certificate Services configuration information, which is read during initial CA installation and whenever the CA certificate is renewed. Some settings are only available for configuration in this file and not using any UI.  
You can use a simple notepad c:\\windows\\capolicy.inf and enter the following text:

```
<pre class="lang:ini decode:true" title="CAPolicy.inf file for Root CA">[Version]
Signature="$Windows NT$"

[Certsrv_Server]
RenewalKeyLength=2048
RenewalValidityPeriod=Years
RenewalValidityPeriodUnits=20
CRLPeriod=years
CRLPeriodUnits=1
CRLDeltaPeriod=Days
CRLDeltaPeriodUnits=0
LoadDefaultTemplates=0
AlternateSignatureAlgorithm=0

[CRLDistributionPoint]
Empty=True

[AuthorityInformationAccess]
Empty=True

```

Where:

- **Renewalkeylength**, **RenewalValidityPeriodUnits** and &lt;**RenewalValidityPeriod** : Only used when you renew your CA certificate. and specify the length of the key and the validity of the CA certificate.
- **CRLPeriod** and **CRLPeriodUnit**&gt; : Specify the frequency of the CA Revocation publication, the period you’re selecting must balance between how often you want to power up the CA to recreate the files and freshness of information you want to insure
- **CRLDistributionPoint** : Leave blank for Root CA, as you don’t have a CRL for such CA.
- **AuthorityInformationAccess**: Leave blank for Root CA for the same reason
- **LoadDefaultTemplates = 0**  prevents CA for publishing and start using default template and cause undesired enrollment. For a stand alone root CA, it is not that important to use it, but for creating suboordinate enterprise CA this is very useful not to automatically advertise every certificate is available on every CA; therefore this line should be your first thought when creating CAPolicy.inf
- **AlternateSignatureAlgorithm = 0** configures the CA to support the PKCS#1 V2.1 signature format for both the CA certificate and certificate requests. Because it is not widely supported, it is often set to 0.

```
<pre class="lang:ps decode:true " title="Root CA installation">Install-AdcsCertificationAuthority -CAType StandaloneRootCA `
-CACommonName "YourCompany Root CA I" `
-CADistinguishedNameSuffix "OU=PKI,O=YourCompany,C=FR" `
-KeyLength 2048 `
-HashAlgorithmName SHA256 `
-CryptoProviderName "RSA#Microsoft Software Key Storage Provider" `
-DatabaseDirectory "D:\CADb\CertDB" `
-LogDirectory "D:\CADb\CertLog" `
-ValidityPeriod Years `
-ValidityPeriodUnits 20
```

Verify that CA is then up and running by issuing:

```
<pre class="lang:batch decode:true " title="Check CA settings from command line">Certutil -CAInfo
Certutil -getreg
```

Please note that after the WIndows Root CA installation, you must still enter a few commands to properly configure it.

## Setting up the characteristics of the issued certificates

If your root CA is going to be for creating issuing CA in a single forest, the following commands set the validity period for the second tier issued certificates. AS you can see the chosen period must be a sub unit of what we choose for the validity of the root CA certificate.

```
<pre class="lang:default decode:true" title="Setting the validity of a issued certificate by the RootCA">certutil -setreg ca\ValidityPeriodUnits 10
certutil -setreg ca\ValidityPeriod Years
net stop certsvc & net start certsvc
```

AS you can see many registry entries only take effect after a restart of the service. If you follow all this guide, you can choose to restart the service at the end of this tutorial.

## Setting the CRL

The following code will set up the CRL so in total it is valid for 6 month.

```
<pre class="lang:batch decode:true " title="Setting the CRL period validity">certutil.exe –setreg CA\CRLPeriodUnits 26
certutil.exe –setreg CA\CRLPeriod Weeks
certutil.exe –setreg CA\CRLDeltaPeriodUnits 0
certutil.exe –setreg CA\CRLDeltaPeriod Days
certutil.exe –setreg CA\CRLOverlapPeriodUnits 12
certutil.exe –setreg CA\CRLOverlapPeriod Hours
```

We are now entering the locations where it will be published:

```
<pre class="lang:ps decode:true" title="Erase Default CRL publishing points">$crls = Get-CACrlDistributionPoint; foreach ($crl in $crls) {Remove-CACrlDistributionPoint $crl.uri -Force};
```

And publishing our own set. PLease note that if you enter this in Powershell, you may want to replace double quotes by simple quotes tmo avoid powershell escaping interpretation

```
<pre class="lang:batch decode:true " title="set the CRL">certutil -setreg CA\CRLPublicationURLs "1:%WINDIR%\system32\CertSrv\CertEnroll\%3%8%9.crl\n10:ldap:///CN=%7%8,CN=%2,CN=CDP,CN=Public Key Services,CN=Services,%6%10\n2:http://pkiweb.mydomain.local/certenroll/%3%8%9.crl"
```

pkiweb.mydomain.local is a web site you’ll make available to your customers. It is optional if the LDAP location is present; however the LDAP Locatoin must not be included if you are going to use the root CA for multiple independent forests, then the web location is mandatory

Don’t forget to run **certutil.exe -crl** command to generate the new CRL. This also checks you syntax about locations.

## Set the AIA

WE now set up the AIA in the same way we set up the CRL

```
<pre class="lang:ps decode:true" title="Erase Default AIA info">$aias = Get-CAAuthorityInformationAccess; foreach ($aia in $aias) {Remove-CAAuthorityInformationAccess $aia.uri -Force};
```

```
```

```
<pre class="lang:batch decode:true " title="set the AIA">certutil -setreg CA\CACertPublicationURLs "1:%WINDIR%\system32\CertSrv\CertEnroll\%1_%3%4.crt\n2:ldap:///CN=%7,CN=AIA,CN=Public Key Services,CN=Services,%6%11\n2:http://pki.contoso.com/certenroll/%1_%3%4.crt"
```

## Auditing

Issue the following command so the CA related events are recorder:

```
<pre class="lang:batch decode:true " title="Set AUdit on CA">certutil -setreg CA\AuditFilter 127
```

## Publish to Active Directory

copy the 2 files in C:\\Windows\\System32\\CertSrv\\CertEnroll to your target forest(s).  
Then log on in each of the target forest(s) with an account that is member of both Domain Admins and Enterprise Admins.

```
<pre class="">certutil.exe –dspublish –f "YourCompany Root CA I.crt" RootCA
certutil –f –dspublish "YourCompany Root CA I.crl"
certutil.exe –addstore –f root "YourCompany Root CA I.crt"
```

The names of the files match the CACommonName display name parameter you used when installing the CA.  
Once you have finished the Windows Root CA installation, you are ready to proceed to the installation to the policy CA or issuing CAs.