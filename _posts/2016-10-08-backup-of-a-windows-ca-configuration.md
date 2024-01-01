---
id: 519
title: 'Backup of a Windows CA Configuration'
date: '2016-10-08T14:47:38+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=519'
permalink: /2016/10/08/backup-of-a-windows-ca-configuration/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"388";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/10/Windows_server_Backup_icon.png
categories:
    - 'ADCS-Certificate Services'
    - PKI
tags:
    - backup
    - CA
    - Certificates
---

In a previous [post](https://dimitri.janczak.net/2016/08/24/windows-root-ca-installation/), I’ve blogged about how to install a Windows root CA, and we did a lot of customization. Let’s see how we can save this configuration using an handy script. What is a backup of a Windows CA Configuration made of?

You may also notice that this script backs up sensitive information which is not always found in the system state you would think it helps you restore everything.

Most commands here are self explanatory:

- First command backs up the Certificate database where all issued, revoked certificates are present
- The second one backs up the private key of the CA. Please note that the system state doesn’t back it up if you don’t apply a hotfix on Windows 2008 R2 as mentioned in this [article](https://support.microsoft.com/en-us/kb/2603469). Of course, if you are using a HSM module, this step is not needed and won’t work.
- The two next commands backup the parameters found in the registry we modified when installing our CA. THe first commands uses the certutil readable format, the second one helps you have an handy registry file to import elsewhere.
- Then we backup the templates we have created and issued
- Finally additional info is stored so when we restore we can compare if we are good to go.

Please note that when restoring, you have to stop the certsvc service and start it again when the modifications are done.

```
<pre class="lang:ps decode:true " title="Backup of CA Configuration using certutil">Function Backup-CAConfig {
param (
[Parameter(Mandatory=$true)]
[string] BackupPath
)
New-Item -Path $BackupPath -ItemType directory
certutil –backupdb $BackupPath
certutil -backupkey $BackupPath
certutil –getreg ca > $BackupPath\CA_certutil_getreg.txt
reg export HKLM\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration $BackupPath\CA_regedir_CertSvcConfiguration.reg
Get-CATemplate|foreach{$_.Name}|out-file -filepath $BackupPath\CATemplates.txt –encoding string –force
certutil –v -catemplates > $BackupPath\Certutil_CATemplates.txt
certutil -cainfo > $BackupPath\Certutil_cainfo.txt
certutil –getreg ca\csp > $BackupPath\certutil_cacsp_getreg.txt
}
```

Some of you may object there is a [backup](https://technet.microsoft.com/en-us/library/cc725565(v=ws.11).aspx) command with certutil but it lacks some of the human readable info outlined here