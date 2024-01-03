---
id: 586
title: 'Could not obtain information about Windows NT group/user &#8216;domain\user&#8217;, error code 0x5'
date: '2017-02-06T16:48:59+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=586'
permalink: /2017/02/06/could-not-obtain-information-about-windows-nt-groupuser-domainuser-error-code-0x5/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:588;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/02/Windows-Authorization-Access-Group.png
categories:
    - 'SQL Server'
tags:
    - 'Active Directory'
    - error
    - T-SQL
    - troubleshooting
    - 'Windows Authorization Access'
---

When getting this error message <span lang="EN-US">Could not obtain information about Windows NT group/user ‘domainname\\username’, error code 0x5 in SQL Server you may be wondering what’s happening.</span>

This error message can happen in numerous occasions and while executing different T-SQL Statements, such as [xp\_logininfo](https://msdn.microsoft.com/en-us/library/ms190369.aspx)

```
<pre class="lang:tsql decode:true" title="xp_loginfo error message">Msg 15404, Level 16, State 19, Procedure xp_logininfo, Line 64
Could not obtain information about Windows NT group/user 'domain\user', error code 0x5.
```

Or while executing [sp\_xp\_cmdshell\_proxy\_account ](https://msdn.microsoft.com/en-us/library/ms190359.aspx)while you are nonetheless syasdmin:

```
<pre class="lang:default decode:true" title="sp_xp_cmdshell_proxy_account error 5">Msg 15137, Level 16, State 1, Procedure sp_xp_cmdshell_proxy_account, Line 1
An error occurred during the execution of sp_xp_cmdshell_proxy_account. Possible reasons: the provided account was invalid or the '##xp_cmdshell_proxy_account##' credential could not be created. Error code: '5'.
```

In all case you may recall that 0x5 means access denied, so the SQL Server service cannot retrieve Active Directory information about the user.

You may then check that

- the user indeed exists in the Active Directory; in particular, are you sure it’s not a SQL Login for which somebody prepended the domain name?
- the SQL Server service account is not a disabled/locked account

If all above fails, you may want to add the service account running SQL Server into the built-in Active Directory Group called Windows Authorization Access Group.

As mentioned in the description, this group has the right to read an attribute on every user object which is not a fixed but computed result of the membership of the global and universal groups.