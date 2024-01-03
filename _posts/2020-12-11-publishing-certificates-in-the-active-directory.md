---
id: 1032
title: 'Publishing certificates in the Active Directory'
date: '2020-12-11T15:13:49+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=1032'
permalink: /2020/12/11/publishing-certificates-in-the-active-directory/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1033;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2020/11/adsiedit-certificatation-authorities-configuration-container.png
categories:
    - 'ADCS-Certificate Services'
    - PKI
    - troubleshooting
tags:
    - ADCS
    - Certificate
    - GPO
    - PKI
    - registry
    - troubleshooting
---

Deploying certificates and CRL in a domain or a forest in an automated fashion can done using GPO like many other settings.

However a less well-known possibility is to use the `certutil -dspublish` command. Let’s review how it works.

When using that option, certificates are stored in one of the “PKI Container”s of the forest, and every time GPO are refresh they are copied into the registry of the computer.

The PKI Settings of a forest are stored under the “Configuration” naming context of the forest.

<figure class="wp-block-image size-large">![ADSI Edit PKI tree](https://dimitri.janczak.net/wp-content/uploads/2020/11/adsiedit-certificatation-authorities-configuration-container.png)<figcaption>Configuration container in AD</figcaption></figure>Under that container, “Certification Authorities” will contain one entry for each Certification Authority Root you want the computers in your forest to trust.

<div class="wp-block-group"><div class="wp-block-group__inner-container is-layout-flow wp-block-group-is-layout-flow">To add a certificate use the following command:

</div></div>```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">certutil -dspublish -f CACertificate.cer RootCA
```

The CACertificate.cer is a file containing the certificate public key to publish in one of the standard X509 formats.

Once you’ve done that, you may want to wait for the Active Directory replication to happen in case the computer you’re testing receives GPOs from another Domain Controller as the one you targeted.

Then wait for the Group Policy refresh to occur (90 minutes more or less in a standard AD environment) or use `gpupdate /force`.

Your certificate authority will be then trusted, as it is now stored in the registry under `HKLM\Software\Microsoft\SystemCertificates\Root\Certificates`.

You’ll find a key with the thumbprint of the certificate and a `blob` entry containing the certificate.

The full list of registry locations that can be targeted with dspublish are [now documented ](https://docs.microsoft.com/en-us/windows/win32/seccrypto/system-store-locations)by Microsoft.