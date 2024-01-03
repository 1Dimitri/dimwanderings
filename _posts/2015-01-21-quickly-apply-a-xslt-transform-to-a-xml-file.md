---
id: 104
title: 'Quickly apply a XSLT Transform to a XML file'
date: '2015-01-21T19:32:24+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=104'
permalink: /2015/01/21/quickly-apply-a-xslt-transform-to-a-xml-file/
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
    - snippet
    - xml
    - xslt
---

Continuing the series of Powershell snippets, here’s how to apply a xslt file to a xml file:

```
<pre class="lang:ps decode:true">$xslt = New-Object System.Xml.Xsl.XslCompiledTransform
$xslt.Load("MyTransform.xsl")
$xslt.Transform("MyXMLFile.xml","MyOutput.WhatEver")
```