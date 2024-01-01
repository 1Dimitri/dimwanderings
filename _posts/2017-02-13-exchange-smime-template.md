---
id: 598
title: 'Exchange S/MIME Template'
date: '2017-02-13T13:56:34+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=598'
permalink: /2017/02/13/exchange-smime-template/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"603";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/02/Outlook-Help-Sign-A-Message.png
categories:
    - Outlook
tags:
    - ADCS
    - Certificate
    - 'certificate template'
    - encryption
    - Exchange
    - 'key archival'
    - S/MIME
    - signing
---

WHen you want to implement mail signing and/or encryption wit the Outlook/Exchange products, you are faced to different choices. One involves to know which Exchange S/MIME template you should choose among all [Certificate templates](https://technet.microsoft.com/en-us/library/cc755033(v=ws.11).aspx).

First of all, please remember that S/MIME may help to achieve the following goals:

- message authentication: this is to be sure that the sender is who (s)he claims to be
- message integrity: so we know the message hasn’t been altered between the sender and the recipient
- message confidentiaility: so the message cannot be seen by any man-in-the-middle

To achieve message authentication and message integrity, you should implement message signing whereas to implement message confidentiality you would implement encryption.

As these capabilities are linked to a user’s email, you would think that the template ‘user’ is the best to fulfill this role. In fact there are multiple reasons which go against this choice:

- The user template is made for several things: Encrypting File System (EFS), Secure Email, Authentication. You do not want necessarily to grant this usage to all users or to grant them with the same certificate  
    [![](https://dimitri.janczak.net/wp-content/uploads/2017/02/User-Certificate-Template-Key-Usage.png)](https://dimitri.janczak.net/wp-content/uploads/2017/02/User-Certificate-Template-Key-Usage.png)
- The AD user object has two fields which may hold certificates: [userCertificate](https://msdn.microsoft.com/en-us/library/cc221422.aspx) and [userSMIMECertificate](https://msdn.microsoft.com/en-us/library/office/hh505703(v=exchg.150).aspx). As above, you may need / want / prefer to separate those roles
- Additionally, remember that you are implementing authentication &amp; message integrity on one side and encryption on the other side. You may also want to separate these roles also.  
    In particular, you may want to be able to decrypt messages if the user is (no longer) available. For this, you would need to do key archival on the certificate used for encryption. But for message signing, key archival makes less sense.

Therefore a best practice is to use:

- Exchange Signature Only for message signing  
    [![](https://dimitri.janczak.net/wp-content/uploads/2017/02/Exchange-Signature-Only-Certificate-Template-Key-Usage.png)](https://dimitri.janczak.net/wp-content/uploads/2017/02/Exchange-Signature-Only-Certificate-Template-Key-Usage.png)
- Exchange User for encryption purposes.

Please note that you should not publish these templates but make copies of them so you can customize duration or CSP/KSP if you’re using a HSM.