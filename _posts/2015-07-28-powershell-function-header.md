---
id: 315
title: 'Powershell Function Header'
date: '2015-07-28T21:18:57+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=315'
permalink: /2015/07/28/powershell-function-header/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"316";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/07/powershell-ise-snippet-list.png
categories:
    - 'Powershell Engine'
tags:
    - documentation
    - function
    - 'function header'
    - ISE
    - powershell
    - snippet
    - template
---

The Powershell ISE may not be your favorite editor because of its limitations (long to start, has issues with high-consuming parallel workflows). However its snippets you can insert with the CTRL+J shortcut may be worthwhile to have a look at. One is particularly interesting: the powershell function header template. [![powershell-ise-snippet-list](http://dimitri.janczak.net/wp-content/uploads/2015/07/powershell-ise-snippet-list.png)](http://dimitri.janczak.net/wp-content/uploads/2015/07/powershell-ise-snippet-list.png)

The function (advanced) has every syntax you can put in the comments and as decorations around parameters when creating new functions. Copying and pasting them in your favorite text editor can save time when editing a powershell function header.

Here is by the way a slightly modified version of the function header so every parameter has all the decorations you can then just erase from to get down to what you want:

```
<pre class="lang:ps decode:true" title="Powershell ISE-like Function header snippet"><#
.Synopsis
   Short description
.DESCRIPTION
   Long description
.EXAMPLE
   Example 1 of how to use this cmdlet
.EXAMPLE
   example 2 of how to use this cmdlet
.INPUTS
   Inputs to this cmdlet (if any)
.OUTPUTS
   Output from this cmdlet (if any)
.NOTES
   General notes
.COMPONENT
   The component this cmdlet belongs to
.ROLE
   The role this cmdlet belongs to
.FUNCTIONALITY
   The functionality that best describes this cmdlet
#>
function Verb-Noun
{
    [CmdletBinding(DefaultParameterSetName='Set 1', 
                  SupportsShouldProcess=$true, 
                  PositionalBinding=$false,
                  HelpUri = 'http://dimitri.janczak.net/',
                  ConfirmImpact='Medium')]
    [OutputType([String])]
    Param
    (
        # Param1 an be named p2 mandatory, 2nd in order,  from pipeline, choose to ValidateXXXX, edit this help text
        [Parameter(Mandatory=$true, 
                   ValueFromPipeline=$true,
                   ValueFromPipelineByPropertyName=$true, 
                   ValueFromRemainingArguments=$false, 
                   Position=0,
                   ParameterSetName='Set 1')]
        [ValidateNotNull()]
        [ValidateNotNullOrEmpty()]
        [AllowNull()]  
        [AllowEmptyCollection()]
        [AllowEmptyString()]
        [ValidateScript({$true})]
        [ValidatePattern("[a-z]*")]
        [ValidateLength(0,15)]
        [ValidateRange(0,100)]
        [ValidateCount(0,5)]
        [ValidateSet("option1", "option2", "option3", "option4")]
        [Alias("p1")] 
        [string[]]  # [int] or  [string] or  [System.Uri] optional type
        $Param1,

        # Param2 can be named p2 mandatory, 2nd in order, but not from pipeline, choose to ValidateXXXX, edit this help text
        [Parameter(Mandatory=$true, 
                   ValueFromPipeline=$false,  # to be erased
                   ValueFromPipelineByPropertyName=$true,  
                   ValueFromRemainingArguments=$false, 
                   Position=1,
                   ParameterSetName='Set 1')]
        [ValidateNotNull()]
        [ValidateNotNullOrEmpty()]
        [AllowNull()]  
        [AllowEmptyCollection()]
        [AllowEmptyString()]
        [ValidateScript({$true})]
        [ValidatePattern("[a-z]*")]
        [ValidateLength(0,15)]
        [ValidateRange(0,100)]
        [ValidateCount(0,5)]
        [ValidateSet("option1", "option2", "option3", "option4")]
        [Alias("p2")] 
        [int]
        $Param2,

        #Param3 simple string mandatory, no alias, no position on the line, Remove Set 2 if not needed, edit this help text
        [Parameter(ParameterSetName='Set 2')]
        [Parameter(Mandatory=$true)]    
        [String]
        $Param3
    )

    Begin
    {
    }
    Process
    {
        if ($pscmdlet.ShouldProcess("Target", "Operation"))
        {
        }
    }
    End
    {
    }
}
```