---
id: 209
title: 'Search for text in a SQL Server database'
date: '2015-06-10T15:26:59+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=209'
permalink: /2015/06/10/search-for-text-in-a-sql-server-database/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:57;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/10/SQL-Server-Logo-no-text.png
categories:
    - 'SQL Server'
tags:
    - database
    - schema
    - search
    - snippet
---

There are many cases when you need to search after some specific text into all the objects of a database:

- replacing references of a commercial product name by another
- moving an application from one environment to another,

Unfortunately not all software developers provide you assistance in this field.

There are many search T-SQL procedures available on the Internet, including [this one](http://blogs.msdn.com/b/developingfordynamicsgp/archive/2013/07/31/updated-spsearchonalldb-sql-stored-procedure-to-search-an-entire-database.aspx) by a former Microsoft guy. However it is not applicable if your database contains multiple schemas. Therefore here is a modified version that will handle those cases. Just create the procedure in the database you want to search for. Except when prefixed with DJ, comments are the original ones.

```
<pre class="lang:tsql decode:true " title="Searching in a SQL Server database whatever the schema">-- Original Works by David Musgrave,
-- at http://blogs.msdn.com/b/developingfordynamicsgp/archive/2013/07/31/updated-spsearchonalldb-sql-stored-procedure-to-search-an-entire-database.aspx
--
-- DJ: 20-May-2015: added support for schemas.

USE [MyTargetDatabase]
GO
IF  EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'[dbo].[spSearchOnAllDB]') AND type in (N'P', N'PC'))
DROP PROCEDURE [dor].[spSearchOnAllDB]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

CREATE PROCEDURE [dbo].[spSearchOnAllDB] @phrase varchar(8000), @OutFullRecords bit = 0 AS

/*
   To apply so: 
      exec  spSearchOnAllDB 'Sugar%'
      exec  spSearchOnAllDB '%soft%'
      exec  spSearchOnAllDB '_5234_57%', 1
      exec  spSearchOnAllDB M_cro_oft
*/

declare @sql varchar(8000)
declare @tbl varchar(128) 
declare @sch varchar(128) 
declare @col varchar(128)
declare @id_present bit

declare @is_char_phrase bit
declare @min_len int
declare @loop_idx int
declare @loop_chr char(1)

set nocount on

if IsNull(@phrase, '') = '' begin
	raiserror('Phrase is absent', 16, -1)
	return
end

-- Handle Quotes passed in the search string
set @phrase = replace(@phrase, '''', '''''')

select @loop_idx = 1, @is_char_phrase = 0, @min_len = 0 

while @loop_idx <= LEN(@phrase) begin
	set @loop_chr = SUBSTRING(@phrase, @loop_idx,1)
	if @loop_chr not in ('%', '_') 
		set @min_len = @min_len + 1
	if @is_char_phrase = 0 and @loop_chr not in ('%', '_', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '.')  
		set @is_char_phrase = 1
	set @loop_idx = @loop_idx + 1
end 

create table #tbl_res 
			(SchemaName	varchar(128) not NULL,
			TableName		varchar(128) not NULL,
			 ColumnName		varchar(128) not NULL,
			 Id				int NULL,
			 ColumnValue	varchar(7500) not NULL)

create table #tbl_res2 
			(SchemaName varchar(128) not NULL,
			TableName		varchar(128) not NULL,
			 ColumnName		varchar(128) not NULL,
			 Id				int NULL,
			 ColumnValue	varchar(7500) not NULL)

declare CRR cursor local fast_forward for
	select  t.schemaname, t.name, c.name, 1 
        -- DJ: add schema
	from (select o.*,s.name as schemaname from sysobjects as o join sys.schemas as s on o.uid=s.schema_id) as  t, syscolumns c
		
	where t.type = 'U'
	and c.id = t.id
	and c.status&0x80 = 0 -- Not IDENTITY
	and exists (select * from syscolumns c2 where t.id = c2.id and c2.status&0x80 = 0x80 and c2.xtype in (48, 52, 56))
	and (  (@is_char_phrase = 1 and c.xtype in (175, 239, 99, 231, 35, 167) and c.length >= @min_len) -- char only
		or (@is_char_phrase = 0 and c.xtype not in (34, 165, 173, 189, 61, 58, 36))) -- char and numeric
	union 
	select  t.schemaname, t.name, c.name, 0   -- DJ: add schema 
		
	from (select o.*,s.name as schemaname from sysobjects as o join sys.schemas as s on o.uid=s.schema_id) as  t, syscolumns c
	
	where t.type = 'U'
	and c.id = t.id
	and not exists (select * from syscolumns c2 where t.id=c2.id and c2.status&0x80 = 0x80 and c2.xtype in (48, 52, 56))
	and (  (@is_char_phrase = 1 and c.xtype in (175, 239, 99, 231, 35, 167) and c.length >= @min_len) -- char only
		or (@is_char_phrase = 0 and c.xtype not in (34, 165, 173, 189, 61, 58, 36))) -- char and numeric
	order by 1, 2

open CRR
fetch CRR into @sch, @tbl, @col, @id_present
while @@FETCH_STATUS = 0 begin
	if @OutFullRecords = 0 begin
		set @sql = 'insert into #tbl_res (SchemaName, TableName, ColumnName, Id, ColumnValue) '
					+ 'select ''[' + @sch + ']'',''[' + @tbl + ']'', ''[' + @col + ']'', '
		if @id_present = 1
			set @sql = @sql + 'IDENTITYCOL, '
		else 
			set @sql = @sql + 'NULL, ' 
		set @sql = @sql + 'convert(varchar(7500), [' + @col + ']) '
						+ 'from [' + @sch + '].[' + @tbl + '] (nolock) '
						+ 'where convert(varchar(8000), [' + @col + ']) like ''' + @phrase + ''' '
	end
	if @OutFullRecords = 1 begin
                -- DJ: display schema
		set @sql = 'if exists (select * from [' + @sch + '].[' + @tbl + '] (nolock) '
				 + 'where convert(varchar(8000), [' + @col + ']) like ''' + @phrase + ''') '
				 + 'select ''[' + @sch + ']'' SchemaName,''[' + @tbl + ']'' TableName, ''[' + @col+ ']'' ColumnName, * '
				 + 'from [' + @sch + '].[' + @tbl + '] (nolock) where convert(varchar(8000), [' + @col + ']) like ''' + @phrase + ''' '
	end
	exec(@sql)
	fetch CRR into @sch,@tbl, @col, @id_present
end
close CRR
deallocate CRR

if @OutFullRecords = 0 begin
	-- For the clients supporting new types:
	--exec('select * from #tbl_res order by 1,2,3')

	-- For the clients who are not supporting new types:
	INSERT #tbl_res2
        -- DJ: add the schema
	select SchemaName, TableName, ColumnName, Id, convert(varchar(255),ColumnValue) ColumnValue from #tbl_res

	/** exec('select TableName, ColumnName, Id, convert(varchar(255),ColumnValue) ColumnValue from #tbl_res order by 1,2,3')**/
end
  
drop table #tbl_res 

/***Select Statement to show tables***/

select SchemaName, TableName, ColumnName, ColumnValue from #tbl_res2 group by SchemaName, TableName, ColumnName, ColumnValue
	order by SchemaName, TableName

truncate table #tbl_res2
drop table #tbl_res2
RETURN
```