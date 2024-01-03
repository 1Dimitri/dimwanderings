---
id: 275
title: 'Download Free Microsoft Ebooks in Powershell'
date: '2015-07-15T20:29:22+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=275'
permalink: /2015/07/15/download-free-microsoft-ebooks-in-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"283";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/07/Powershell-File-Downloader-Write-Progress2.png
categories:
    - 'Powershell Engine'
tags:
    - download
    - ebook
    - Invoke-WebRequest
---

As mentioned [previously](http://dimitri.janczak.net/2014/08/10/invoke-webrequest-prequel-alternative-system-net-webclient/), Powershell 3.0 introduces a cmdlet called Invoke-WebRequest that allows do script your web exchanges, as it allows to retrieve URLs, fill forms back, and easily analyze any HTML output.

One usage of such a cmdlet is to download documents. However many examples on the web do not take into account the multiple URL shorteners and other redirection mechanisms the web is filled with nowadays.

Our example is the well-known list of [hundred free ebooks ](http://blogs.msdn.com/b/mssmallbiz/archive/2015/07/07/i-m-giving-away-millions-of-free-microsoft-ebooks-again-including-windows-10-windows-8-1-windows-8-windows-7-office-2013-office-365-sharepoint-2013-dynamics-crm-powershell-exchange-server-lync-2013-system-center-azure-clo.aspx)that authors and editors for Microsoft Press have released. In the blog post, the author says he won’t provide you with a zip file but let you download a list of links. Let’s retrieve that file and see how to work with Invoke-WebRequest.

In a candid manner, we could think about this simple script

```
<pre class="lang:ps decode:true" title="Simplistic File Download  Powershell script">Import-CSV 'MSFTEbooks2.txt' | { iwr $_.'MSFT eBooks' -outfile 'somename.pdf' }
```

As you have already noted, we run into trouble to know under which name we should save each link, because it is not in the link name. As Firefox, Internet Explorer and other browsers manage to get the final name, this information is nonetheless somewhere encoded during the multiple redirections. If you know basic HTML, you’ve probably have the idea to look at the headers of the web responses.

Using one of the web developer tools (fireBug, etc.) , we can look into those headers and see that location will be our friend here:

[![Firebug-NetPanel-Redirection-Header](http://dimitri.janczak.net/wp-content/uploads/2015/07/Firebug-NetPanel-Redirection-Header.png)](http://dimitri.janczak.net/wp-content/uploads/2015/07/Firebug-NetPanel-Redirection-Header.png)

Unfortunately we’ll encounter two problems here

- as mentioned in the cmdlet documentation, if you use outfile you cannot retrieve the headers unless you use PassThru, so we will use the pipeline and save the file ourselves
- we’ll soon discover that location is not available if we use such a code snippet:

```
<pre class="lang:ps decode:true " title="Looking at Web reponse headers in Powershell">$request = iwr $link 
Write-Host "Location is [$($request.headers.location)]"
```

Indeed if we look back at trace, the location is only present during the various redirections we encounter and disappear for the last response that Invoke-WebRequest sent back to us. A look at the [cmdlet documentation](https://technet.microsoft.com/en-us/library/hh849901.aspx) will give us a hint with the MaxRedirection argument: we can indeed tell that cmdlet to follow a given number of redirections.

Sadly, we don’t know that number, just that we want them all but the last one. Therefore we’ll have to do it manually by checking each response to see if it is still a redirection or not

```
<pre class="lang:ps decode:true " title="Manually following redirections with Invoke-WebRequest">$link = "http://server/link"
do {
$request = Invoke-WebRequest -Uri $link -Max 0 -EA SilentlyContinue
if ($request.headers.location) { $link=$request.headers.location }
} while (($request.StatusCode -gt 300) -and ($request.StatusCode -lt 400))
```

The -EA (ErrorAction) argument is just here to suppress the nasty red error message about intermediate unfinished redirections.

The last two tips for the final code snippet are:

- Use of \[System.Uri\] object to transform the last segment of the retrieved URL into a viable filename
- Use of \[System.IO.File\]::WriteAllBytes to quickly save the contents we have in memory instead of a long Set-Content -Encoding Byte piece of code

```
<pre class="lang:ps decode:true " title="Multiple File Downloader For MSFT Ebook">$percent = 0
Write-Progress -Activity "Reading list..." -PercentComplete $percent -CurrentOperation "$percent% complete"  -Status "Please wait."
$list = Import-CSV 'MSFTEbooks2.txt'
$percentstep = 100  / $list.count
$list | % {
$Initiallink = $_.'MSFT eBooks'
$tempname = [System.IO.Path]::GetTempFileName()
Write-Progress -Activity "Getting $Initiallink" -PercentComplete $percent -CurrentOperation "Resolving $Initiallink"  -Status "Please wait."
$previousheaders = $null
$percent+=$percentstep
$maxredirs = 0
$ev = $null
$link=$initiallink
do {
Write-Progress -Activity "Getting $Initiallink" -PercentComplete $percent -CurrentOperation "Resolving $link"  -Status "Please wait."
$request = $null
$request = Invoke-WebRequest -Uri $link -Max 0 -EA SilentlyContinue
if ($request.headers.location) { $link=$request.headers.location }
} while (($request.StatusCode -gt 300) -and ($request.StatusCode -lt 400))
$bshortname = [System.Uri]::UnescapeDataString(([System.URI]$link).Segments[-1])
$targetname = Join-Path (Get-Location).Path $bshortname
Write-Progress -Activity "Getting $Initiallink" -PercentComplete $percent -CurrentOperation "Saving as $targetname"  -Status "Please wait."
[System.IO.File]::WriteAllBytes($targetname,$request.content)
$percent+=$percentstep 
}
$percent = 100
Write-Progress -Activity "Finished" -PercentComplete $percent -CurrentOperation "Books saved in $(Get-Location)"  -Status "Done"
```