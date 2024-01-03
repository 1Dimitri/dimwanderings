---
id: 1101
title: 'Length of a SQL Server instance name'
date: '2021-08-17T15:16:32+01:00'
author: Dimitri
excerpt: 'What is the length of a SQL Server instance name, the invalid characters, and also for a Windows machine machine'
layout: article
guid: 'https://dimitri.janczak.net/?p=1101'
permalink: /2021/08/17/length-of-a-sql-server-instance-name/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1103;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2021/08/System-Properties-Windows.jpg
categories:
    - 'SQL Server'
    - 'Windows OS'
tags:
    - 'server name'
---

Recently a colleague of mine in charge of an inventory tool asked me what is the maximum length of a SQL Server instance name.

So let’s dig into the various locations where Microsoft speaks about length of names in Windows

First stop, the SQL Server documentation itself. A quick search conducts us to this link in the [legacy documentation of SQL Server 2008 R2](https://docs.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms143531(v=sql.105))

There’s a good explanation of the instance part, aka. the name part after the backslash if your instance is named. Short reminder: if that part is not mentioned it is supposed to be MSSQLSERVER, but that doesn’t give us the upper limit for the number of characters. That part is limited to 16 characters, must start with a letter, but subsequent characters can be made of letters, digits, the dollar sign or the underscore. Some other punctuation signs are explicitly forbidden such as space, comma, semi-colon, colon, ampersand, single quote, at sign, but other are in the gray area as not mentioned: what about double-quotes for example?

For the server part, things are a bit more complicated as we are depending on the OS limits. On Windows, there is an article about the [machine naming limits](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou).

Here the legacy NetBIOS name is a 1 to 15 character field, but the list of invalid characters interestingly does not comprise the “dot”, nor the “space”, just the usual filename invalid character suspects: backslash, slash, colon, asterisk, question mark, double quote, pipe sign, less and greater than sign mark.

If you go for the DNS name, you can extend the limit to 255 characters by 63 bytes chunks but the list of invalid characters is first defined by what is allowed: letters, digits, minus sign. Do not forget that officially the underscore is not supported albeit Microsoft DNS Servers if configured allow it and it is listed as disallowed character in that same article,

To summarize, if I had to store SQL Server instance names into databases into single tenant environments, I would probably choose 32 characters, by adding up the NetBIOS server name limit of 15 characters, the backslash and the 16 characters limit of the instance name part.

In multiple tenant environments, I may choose to widen the server name part to 255 characters to store the FQDN as given by the DNS.