---
id: 26
title: 'Invoke-WebRequest prequel alternative: System.Net.WebClient'
date: '2014-08-10T10:46:59+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=26'
permalink: /2014/08/10/invoke-webrequest-prequel-alternative-system-net-webclient/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"18";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/08/powershell-logo-e1410861403696.jpg
categories:
    - 'Powershell Engine'
tags:
    - curl
    - Invoke-WebRequest
    - 'web requests'
---

The newest version of Powershell, 4.0, includes the [Invoke-WebRequest](http://technet.microsoft.com/en-us/library/hh849901.aspx "Invoke-WebRequest") cmdlet to retrieve data using the http protocol and its derivatives.  
If you do not work for Microsoft and because of the systems you are facing to, need to be compatible with previous versions, the following one-liner can easily replace it for simple-regular downloads (GET requests)

```
<pre class="lang:ps decode:true " title="System.net.WebClient usage">(New-Object System.Net.WebClient).DownloadString("http://your-url")
```

The contents will be retrieved in a string. If you need to handle a binary file, the DownloadData method is then your friend.