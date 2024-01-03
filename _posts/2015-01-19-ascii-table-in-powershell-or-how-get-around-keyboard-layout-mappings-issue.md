---
id: 97
title: 'ASCII Table in Powershell (or how get around keyboard layout mapping issues)'
date: '2015-01-19T20:47:18+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=97'
permalink: /2015/01/19/ascii-table-in-powershell-or-how-get-around-keyboard-layout-mappings-issue/
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
    - 'ascii table'
    - 'citrix receiver'
    - 'keyboard layout'
    - snippet
---

Imagine you have to connect to a Citrix server using the java client and the keyboqrd layout of the machine you’re using is not properly sert up there?

Imagine that the CItrix administrator doesn’t have the character map tool enabled.

Just use this snippet of powerhell to get a nice character list in a powershell window:

```
<pre class="lang:ps decode:true " title="ASCII Table">-join (32..127 |  % { [Char] $_ })
```

Notice how the join operator can be used with a empty left operator to concatenate strings on the same line.