---
id: 1019
title: 'Who&#8217;s making your log file grow in SQL Server?'
date: '2020-07-06T15:55:36+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=1019'
permalink: /2020/07/06/whos-making-your-log-file-grow-in-sql-server/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1022;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2020/07/AdventuresInSQLServer-repo.jpg
categories:
    - 'SQL Server code'
    - Uncategorized
tags:
    - DMV
    - 'log file'
    - troubleshooting
---

Determining what T-SQL query or SQL Server client application is the source of log file growth is an often asked question.

“Old” SQL DBAs would sometimes refer you to [KB 317375](https://support.microsoft.com/en-us/help/317375/a-transaction-log-grows-unexpectedly-or-becomes-full-in-sql-server) which unfortunately is no longer hosted as a Microsoft KB Article, but sadly infinitely loops on one new [Docs @ Microsoft page](https://docs.microsoft.com/en-us/sql/relational-databases/logs/troubleshoot-a-full-transaction-log-sql-server-error-9002?view=sql-server-ver15). Note also that the way the support page at Microsoft was built creates also issue for the [WayBack Machine to di](http://web.archive.org/web/20190201080033/https://support.microsoft.com/en-us/help/317375/a-transaction-log-grows-unexpectedly-or-becomes-full-in-sql-server/)splay.

In order to keep a trace on how to proceed in such a case, let’s store in this article a few points on how to tackle this.

- Since SQL Server 2005, the answer is somewhere in the [Dynamic Management Views](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views?view=sql-server-ver15). The goal is to roughly know which DMVs may be helpful here. Only objection granted to the reader: since SQL Server 2008, may be some Extended Event may also help you.
- In our case, the answer will be stored somewhere in the row of the [sys.dm\_tran\_database\_transactions](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-database-transactions-transact-sql?view=sql-server-ver15) DMV. Fields of interest start with database\_transaction\_log\_bytes\_\*
- Unfortunately this view is not enough as the transaction in the database contains just an ID and not the query itself.
- To get the query we have to join the database transactions view on the [session transactions ](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-session-transactions-transact-sql?view=sql-server-ver15)view. From there, the session id of the session transactions view will allow us to get the T-SQL text in the [dynamic management function sql\_text](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sql-text-transact-sql?view=sql-server-ver15) by joining on the [requests view](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql?view=sql-server-ver15).
- Notice that the sqltext field doesn’t give you the precise statement you’re hunting, but may give you the full text of a stored procedure if this statement is a part of such an object. You must use statement\_start\_offset and statement\_end\_offset to calculate the true “sub-statement” if it exists.

All those directions were more or less explained in the original article but the ready-to-use in-case-of-emergency full T-SQL solution was not part of this article nor it is part of the newer docs page. The ready to use statement is as follows:

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="sql" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">SELECT sesstran.session_id AS [spid]
, DB_NAME(dbtran.database_id) AS [dbname]
, QUOTENAME(DB_NAME(sqltxt.dbid)) + N'.' + QUOTENAME(OBJECT_SCHEMA_NAME(sqltxt.objectid, sqltxt.dbid)) + N'.' + QUOTENAME(OBJECT_NAME(sqltxt.objectid, sqltxt.dbid)) AS sql_object
, req.command as [sql_command]
, SUBSTRING(sqltxt.text, ( req.statement_start_offset / 2 ) + 1
, ((CASE req.statement_end_offset 
WHEN -1 THEN DATALENGTH(sqltxt.text) 
ELSE req.statement_end_offset 
END - req.statement_start_offset)/2)+1) AS [sql_command_param]
, dbtran.database_transaction_log_bytes_used / 1048576.0 AS [log_used_mb]
, dbtran.database_transaction_log_bytes_used_system / 1048576.0  AS [logsystem_used_mb]
, dbtran.database_transaction_log_bytes_reserved / 1048576.0 AS  [log_reserved_mb]
, dbtran.database_transaction_log_bytes_reserved_system / 1048576.0 AS [logsystem_reserved_mb]
, dbtran.database_transaction_log_record_count AS [log_records]
FROM sys.dm_tran_database_transactions dbtran 
JOIN sys.dm_tran_session_transactions sesstran ON dbtran.transaction_id = sesstran.transaction_id 
JOIN sys.dm_exec_requests req 
CROSS apply sys.dm_exec_sql_text(req.sql_handle) AS sqltxt
ON sesstran.session_id = req.session_id 
ORDER BY 5 DESC; 
```

This snippet is part of the repository of various tools I store in a [SQL Server related Github repository](https://github.com/1Dimitri/AdventuresInSQLServer/blob/master/files/WhosGeneratingLog.sql).