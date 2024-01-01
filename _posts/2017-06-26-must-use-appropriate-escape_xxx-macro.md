---
id: 737
title: 'you must use the appropriate ESCAPE_xxx macro'
date: '2017-06-26T14:12:49+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=737'
permalink: /2017/06/26/must-use-appropriate-escape_xxx-macro/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:738;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/06/SQL-Job-History-Escape_xxx_macro_error.png
categories:
    - 'SQL Server'
tags:
    - code
    - collation
    - 'sql agent jobs'
    - 'sql server agent'
    - troubleshooting
---

Under some circumstances, your SQL Server Agent job mail fails with the following error message: “Unable to start execution of step 1 (reason: The job step contains tokens. For SQL Server 2005 Service Pack 1 or later, you must use the appropriate ESCAPE\_xxx macro to update job steps containing tokens before the job can run.). The step failed.”

Obviously if you have jobs you created a long long time ago, you may well be finding yourself in the situation the message describes, i.e. upgrading to SQL Server 2008, 2008R2, 2012, 2012, 2014 or 2016.

However sometimes this message also appears when porting existing jobs from existing SQL Server instances to the same version or roughly equivalent one. (e.g. SQL Server 2012 SP3 to SQL Server 2016 SP1). Indeed the error may occur because you are moving from a case insensitive collation instance to a case sensitive or binary code point one. The problem lies into the $VAR sequence which are in use in the job steps.

The solution outlined in [KB 915845: SQL Server Agent jobs fail when the jobs contain job steps that use tokens after you install SQL Server 2005 Service Pack 1](https://support.microsoft.com/en-us/help/915845/sql-server-agent-jobs-fail-when-the-jobs-contain-job-steps-that-use-tokens-after-you-install-sql-server-2005-service-pack-1) is however still valid and useful.

You must just:

1. run the code available in the KB article
2. run the created stored procedure in msdb to update one or all the jobs ```
    <pre class="lang:tsql decode:true" title="running the stored procedure in KB 915845">exec msdb.dbo.sp_AddEscapeNoneToJobStepTokens 'Display name of the job' -- specific job
    exec msdb.dbo.sp_AddEscapeNoneToJobStepTokens -- all jobs
    ```

If you wish to go further into understanding this ESCAPE\_xxx macro, you can get a quick explanation on the page [Use Tokens in Job Steps](https://docs.microsoft.com/en-us/sql/ssms/agent/use-tokens-in-job-steps).