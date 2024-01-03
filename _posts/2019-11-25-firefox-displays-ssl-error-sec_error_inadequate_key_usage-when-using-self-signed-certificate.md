---
id: 987
title: 'Firefox displays SSL Error: SEC_ERROR_INADEQUATE_KEY_USAGE when using self-signed certificate'
date: '2019-11-25T16:14:43+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=987'
permalink: /2019/11/25/firefox-displays-ssl-error-sec_error_inadequate_key_usage-when-using-self-signed-certificate/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:988;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2019/11/Firefox-Preferences-Privacy-Certificates.png
categories:
    - 'troubleshooting tips'
tags:
    - Certificate
    - firefox
    - PKI
    - yunohost
---

When using self-signed certificates, and accessing your web contents with the Firefox browser, you may get a strange error message SEC\_ERROR\_INADEQUATE\_KEY\_USAGE

This may happen in self hosted instances of YunoHost for example.

Whereas the contents is properly displayed in all other browsers, you cannot add any exception in Firefox and may think there’s an error in the generated certificate.

But the message is misleading. As[ explained in this thread](https://bugzilla.mozilla.org/show_bug.cgi?id=1590217), this is not a matter of certificate generation but a matter of CA Chain resolution algorithm as implemented by Firefox.

The culprit is in the name of the Certification Authority which matches the name of one of the website for which the certificate has been issued.

There are 2 solutions :

- Either you can change the Distinguished Name of the CA
- Or you must add the CA root certificate to the list of trusted CAs within Firefox. Please note that it must be put in the Firefox store; using the Windows computer store for example won’t work. The menu to import the certificate is available by typing “about:preferences#privacy” in the address bar, and using the “view certificates” button.