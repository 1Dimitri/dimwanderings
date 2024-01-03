---
id: 561
title: 'Regional Independent date time batch job'
date: '2016-12-20T16:33:41+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=561'
permalink: /2016/12/20/regional-independent-date-time-batch-job/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"562";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/12/date-time-panel.jpg
categories:
    - batch
tags:
    - date
    - snippet
    - time
---

Got pissed off because your batch file has to know the current date and time and depending on the user or machine logged on you don’t get the same result? Let’s use today snippet: regional independent date time batch job.

In the past, you used:

- date /t
- time /t

But did you know there was a [Win32\_LocalTime](https://msdn.microsoft.com/en-us/library/aa394171(v=vs.85).aspx) which exposes date parts and time parts as properties?

Don’t know how to use that class? Look at that snippet:

```
<pre class="lang:batch decode:true " title="Get Date and Time irrespective of regional settings">FOR /F "skip=2 tokens=2-7 delims=," %%A IN ('WMIC Path Win32_LocalTime Get Day^,Hour^,Minute^,Month^,Second^,Year /Format:csv') DO ( 
       SET Today=%%F%%D%%A 
       SET Now=%%B%%C%%E 
) 
```

You may change the format of date stored in today and the time in now by changing the order of the tokens’ variables.

Feel free to replace WIn32\_LocalTime by Win32\_UTCTime if you need to be time zone independent