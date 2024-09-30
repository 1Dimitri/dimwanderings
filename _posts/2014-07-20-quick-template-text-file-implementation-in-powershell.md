---
id: 28
title: 'Quick template text file implementation in Powershell'
date: '2014-07-20T22:17:46+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=28'
permalink: /2014/07/20/quick-template-text-file-implementation-in-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";b:0;s:11:"_thumb_type";s:10:"attachment";}'
categories:
    - 'Powershell Engine'
tags:
    - evaluation
    - template
---

If you write scripts, there are good chances you have some of them whose sole purpose is to duplicate configuration files or reports with data depending on variables.  
A quick way to implement this in Powershell is to use the combined power of [Here-Strings](https://en.wikipedia.org/wiki/Here_document#Windows_PowerShell "Here Strings") and [expressions](http://technet.microsoft.com/en-us/library/hh847892.aspx "Powershell expression vs. command mode") using variables.

The basic idea is to store the template as a text file, using variables and derived expressions to act as data placeholders :

```
<pre class="lang:default decode:true ">This is my template

I put data from simple variable content : $name, $first
I put data from expression to timestamp the output : $(Get-Date)
And I even add calculations if needed $($mb/1MB) MB 

```

Then we’ll have to figure out how to read that template file, let Powershell replace the variables and expressions by their contents, and do whatever we need.

If you simply read the contents of the file with Get-Content, the variables are not expanded, so you’ll have to figure out how to build an instruction, expression, or scriptblock that will make Powershell interpret it.

Since we are talking about expressions, let’s use Invoke-Expression. The only missing piece of the puzzle is how to transform that string (or array of strings) we read with Get-Content into a valid Powershell expression.

Here (!) come the Here-Strings by noticing that the following code will build a valid here-string from any variable

```
"@`"`r`n$variable`r`n`"@"
```

If you have trouble reading that expression, remember that

- the opening and ending tags of a here-string must be on their own line, hence the CR LF pairs;
- the CR LF, and double-quote symbol must be written using the backtick as we are inside double-quoted string.

Then our full solution will be:

```
<pre class="lang:default decode:true " title="Text Templates in Powershell">$template = Get-Content 'mytemplate.txt' -Raw
$expanded = Invoke-Expression "@`"`r`n$template`r`n`"@"
```
