---
id: 154
title: 'Jobs in Powershell'
date: '2015-05-18T19:55:22+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=154'
permalink: /2015/05/18/jobs-in-powershell/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"18";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2014/08/powershell-logo-e1410861403696.jpg
categories:
    - 'Powershell Engine'
tags:
    - cdxml
    - cheatsheet
    - jobs
    - reference
    - remoting
    - wmi
---

This post serves more as a reference that an introduction to Powershell Jobs.

- Cmdlets are: **Get-Job**, **Receive-Job, Remove-Job**, **Resume-Job, Start-Job, Stop-Job** , **Suspend-Job** and **Wait-Job** but other commands have -AsJob parameters (Get-WMIObject, Invoke-Command, CDXML snippets)
- Help can be found in the following topics: 
    - about\_Jobs
    - about\_Job\_Details
    - about\_Remote\_Jobs
    - about\_Scheduled\_Jobs
    - about\_Scheduled\_Jobs\_Advanced
    - about\_Scheduled\_Jobs\_Basics
    - about\_Scheduled\_Jobs\_Troubleshooting
- **Suspend-Job** and **Resume-Job** are only available with workflows
- **Receive-Job** deletes the data it sends to you except if you use **-Keep** as a parameter
- PSJobTypeName can be 
    - BackgroundJob (regular Start-Job),
    - WMIJob (Get-WmiObject -AsJob),
    - CimJob (use CDXML files, not Get-CimInstance),
    - PSScheduledJob (Register-ScheduledJob)
    - PSWorkflowJob (nameofmyworkflow -AsJob)
- As a job is run in another Windows Powershell process than yours, you cannot use it with cmdlets such as **Register-WMIEvent**
- **Start-Job** runs the command locally (but that command may perform remote connections), whereas **Invoke-Command** runs the command remotely and you get the response back.
- If you use parameter(s), you must use ArgumentList to pass them around as jobs are run in separate Powershell instances and processes (unsecapp for Get-WMIObject, wsmprovhost for Invoke-Command)
- Sessions (via Enter-PSSession or New-PSSession) are a way to maintain that remote connection when you have multiple script blocks to run on a target machine
- ScheduledTasks is a CDXML (WMI Wrapper) module about the TaskScheduler, whereas PSScheduledJob manages the Powershell Scheduled Jobs which are… tasks found under \\Microsoft\\Windows\\Powershell\\ScheduledJobs
- ScheduledJobs are primarily maintained with Register-ScheduledJob, but to work with result you must import explicitly the module and you can only see the jobs that you create/manage in your context whose results are stored in %AppData%\\Local\\Microsoft\\Windows\\Powershell\\ScheduledJobs\\&lt;JobName&gt;\\Output
- If you run jobs inside workflows, you lose the result because they are still run in an external process to yours
- Some snippets: ```
    <pre class="lang:ps decode:true" title="Powershell Jobs examples"># Simple
    Start-Job -ScriptBlock { ... some action ...} -Name MySupremeJob
    Get-Job -Name MySupremeJob | select PSBeginTime, PSEndTime, @{Name='Duration';Expression={$_.PSEndTime-$_.PSBeginTime}}
    # with parameter
    Start-Job -Scriptblock {param([string]$data) Get-Service -Name $data} -ArgumentList $data
    
    #WMI Method
    $stuckjava = Get-WMIObject -Class Win32_Process -Filter "Name='javaw.exe'"
    Invoke-WmiMethod -InputObject $stuckjava -Name Terminate -AsJob
    # ScheduledJOb
    $everyday = New-JobTrigger -Daily -At "23:59"
    $god=New-ScheduledOption -RunElevated
    Register-ScheduledJob -Name RebootMe -ScritpBlock {Restart-Computer} -Trigger $everyday -ScheduledJobOption $god
    #workflows
    workflow donothing { Start-Sleep -Seconds 60; Suspend-Workflow; Write-Host "Waking up myself" } #auto-suspend
    workflow stilldonothing { Start-Sleep -Seconds 60; Checkpoint-Workflow; Write-Host "Waked up" } #suspend the job from elsewhere
    #Remote Sessions launching jobs
    $faraway = New-PSSession -ComputerName mydistantpc
    Invoke-Command -Session $faraway -ScriptBlock {Start-Job -Name WhichSvc -Scriptblock { Get-Service } }
    Invoke-Command -Session $faraway -ScriptBlock {Receive-Job -Name WhichSvc -Keep }
    Invoke-Command -Session $faraway -ScriptBlock {Get-Job | Remove-Job }
    $faraway | Remove-PSSession
    
    
    ```