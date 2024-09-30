---
id: 17
title: 'Wrapping an existing command Send-MailMessage using splatting'
date: '2014-07-19T19:32:19+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=17'
permalink: /2014/07/19/wrapping-an-existing-command-send-mailmessage-using-splatting/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"32";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/07/Windows-PowerShell-Logo.png
categories:
    - 'Powershell Engine'
tags:
    - mail
    - 'powershell function'
    - 'powershell metadata'
    - send-mailmessage
    - smtp
---

Here is a simple way to add parameters to an existing command when you do not need nor want to overwrite the function name:

- Create a new function
- Add the new parameter(s) at the beginning of the parameter list
- use “ValueFromRemainingArguments” to collect the parameters you want to send to the original cmdlet; you can name the parameter as you want
- When calling the original cmdlet, use the splatting operator “@” with the parameter name you put last in the parameter list.

Here the example wraps the Send-MailMessage cmdlet in case the SMTP Server is not available, etc in a try/catch blog and whatever the result logs all parameters and the current timestamp into a CliXml file for future handling or retry.

```
Function Test-SendMailMessage
{
  [CmdletBinding()]
    Param
    (
        [parameter(mandatory=$true, position=0)][string]$LogFile,
        [parameter(mandatory=$false, position=1, ValueFromRemainingArguments=$true)]$MyArgs
    )
 

 try
 {
	Write-Verbose "Trying to send a message"
	Send-MailMessage @myargs
 }
 catch
 {
	Write-Verbose "Unable to send message"
  }
  Write-Verbose "Storing output in $LogFile"
 Export-CliXml  -Path $LogFile -InputObject $myArgs,(Get-Date)
}
```