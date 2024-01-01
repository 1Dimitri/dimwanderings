---
id: 851
title: 'Powershell criticism part 1'
date: '2018-04-09T11:57:05+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=851'
permalink: /2018/04/09/powershell-criticism-part-1/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:855;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/04/New-SmbMapping-Credential-Get-Help.png
categories:
    - 'Powershell Engine'
tags:
    - cleartext
    - cmdlet
    - language
    - New-SmbMapping
    - ranting
    - security
---

In a [Tweet](https://twitter.com/jsnover/status/948270117283958784), Jeffrey Snover was asking if the community had anything to say to make Powershell better. This kind of tweet may not be very easy to answer in 280 characters, but if you take time to think about it, it interrogates the way you’re using the language. Did you in fact make your scripts powershellic ? Do you have issues with some edge cases of the language grammar ? Are some various commonly used cmdlets not well aligned with the language’s spirit, whether they are built-in or comes from well-known packages.

Before digging into the language in a future post, let’s start first by a very annoying cmdlet for the system administrator: [New-SmbMapping](https://docs.microsoft.com/en-us/powershell/module/smbshare/new-smbmapping?view=win10-ps). Whether your sysadmin experiment started before or with Powershell, this cmdlet does not align with the rest of the commands you’re used to run. The issue lies in the management of the credentials, as you often use alternate credentials to connect to the target share.

If you are Powershell savvy, credentials mean [PSCredential](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential), so you’ll be surprised the first time you try to auto-complete this cmdlet, when nothing comes up which starts with ‘C’.

[![](https://dimitri.janczak.net/wp-content/uploads/2018/04/New-SmbMapping-Credential-Unknown-parameter.png)](https://dimitri.janczak.net/wp-content/uploads/2018/04/New-SmbMapping-Credential-Unknown-parameter.png)

You’ll soon discover that the two options to use are UserName and Password.

[![](https://dimitri.janczak.net/wp-content/uploads/2018/04/New-SmbMapping-Credential-Get-Help.png)](https://dimitri.janczak.net/wp-content/uploads/2018/04/New-SmbMapping-Credential-Get-Help.png)

You wonder then why you were told to always use[ Get-Credential](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-credential) to request sensitive information, and why here your password becomes a simple string, especially when in comparison you’re warned when [doing the same plain text password thing in Desired State COnfiguration files](https://blogs.technet.microsoft.com/ashleymcglone/2015/12/18/using-credentials-with-psdscallowplaintextpassword-and-psdscallowdomainuser-in-powershell-dsc-configuration-data/)

But that’s not the first surprise you will encounter. Naively you tell yourself that this parameter naming convention is used because the cmdlet mimics the legacy net use command. Therefore you try to input only a username to get prompted for a password. Doesn’t work.

[![](https://dimitri.janczak.net/wp-content/uploads/2018/04/New-SmbMapping-Credential-No-Password-Prompt.png)](https://dimitri.janczak.net/wp-content/uploads/2018/04/New-SmbMapping-Credential-No-Password-Prompt.png)

Familiar with similar but not equivalent net related commands, you try to use a star as a password. Doesn’t work either. You’ll get the same error. Since criticism without enhancement proposal would be useless, here is a modified version of New-SmbMapping which proxies the current existing version.

```
<pre class="lang:ps decode:true" title="New-SmbMapping Credential version">unction New-SmbMappingEx {
[CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='Medium', PositionalBinding=$false, HelpUri='http://go.microsoft.com/fwlink/?LinkID=241956')]
param(
    [Parameter(ParameterSetName='Create0', Position=1)]
    [ValidateNotNull()]
    [AllowEmptyString()]
    [string]
    ${LocalPath},

    [Parameter(ParameterSetName='Create0', Position=2)]
    [ValidateNotNull()]
    [AllowEmptyString()]
    [string]
    ${RemotePath},

    [Parameter(ParameterSetName='Create0')]
    [PSCredential]
    ${Credential},

    [Parameter(ParameterSetName='Create0')]
    [bool]
    ${Persistent},

    [Parameter(ParameterSetName='Create0')]
    [switch]
    ${SaveCredentials},

    [Parameter(ParameterSetName='Create0')]
    [switch]
    ${HomeFolder},

    [Parameter(ParameterSetName='Create0')]
    [Alias('Session')]
    [ValidateNotNullOrEmpty()]
    [CimSession[]]
    ${CimSession},

    [Parameter(ParameterSetName='Create0')]
    [int]
    ${ThrottleLimit},

    [Parameter(ParameterSetName='Create0')]
    [switch]
    ${AsJob})

begin
{
    try {
        $outBuffer = $null
        if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
        {
            $PSBoundParameters['OutBuffer'] = 1
        }




        $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('New-SmbMapping', [System.Management.Automation.CommandTypes]::Function)

 
        if ($PSBoundParameters.ContainsKey('Credential')) {
             [void] $PSBoundParameters.Remove('Credential')
             $scriptCmd = {& $wrappedCmd @PSBoundParameters -Password ($credential.GetNetworkCredential().Password) -Username $credential.Username }
        } else {
            $scriptCmd = {& $wrappedCmd @PSBoundParameters }
        }

       
        $steppablePipeline = $scriptCmd.GetSteppablePipeline()
        $steppablePipeline.Begin($PSCmdlet)
    } catch {
        throw
    }
}

process
{
    try {
        $steppablePipeline.Process($_)
    } catch {
        throw
    }
}

end
{
    try {
        $steppablePipeline.End()
    } catch {
        throw
    }
}

}
<#

.ForwardHelpTargetName New-SmbMapping
.ForwardHelpCategory Function

#>
```

Here is a proxy command which uses Credential as

After this post about a Microsoft issued cmdlet not following Microsoft best practices, next post will discuss a language construction.