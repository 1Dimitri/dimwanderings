---
id: 1106
title: 'Are Windows Event Logs displaying local time or UTC? There&#8217;s a trap!'
date: '2021-08-24T14:25:25+01:00'
author: Dimitri
excerpt: 'Is Local time or UTC used in Windows Event Log. Both, it depends where'
layout: post
guid: 'https://dimitri.janczak.net/?p=1106'
permalink: /2021/08/24/are-windows-event-logs-displaying-local-time-or-utc-theres-a-trap/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:1107;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2021/08/Powershell-Time-Zone-EventLog.png
categories:
    - 'Windows OS'
tags:
    - 'event log'
    - time
    - timezone
---

Often [the question arises to know if the time stamps in the Windows Event Logs are local or UTC](https://serverfault.com/questions/586250/what-time-zone-is-displayed-in-windows-event-logs-when-viewing-saved-log-from-a). The answer is more subtle than that. Let’s discuss which time zone is used in the Windows Event Logs and where.

First of all, please note that this discussion apply to the “Operating System Event Logs” you can view with the Event Viewer, such as Application, System, Security and the newer Vista-like Logs such as CAPI/Operational. It does not apply to the various “.log” files you can find in the temp folders, such as the installation traces of Visual Studio.

If you look at your PC or server right now, you will see that the events seem displayed with your local time. But beware This is only valid for the field “Date and Time” as the timestamp is recorded in fact as UTC and the offset is calculated by the client application.

However anything else is just text and is recorded as such so it is recorded by using the timezone of the application which recorded the message. Sounds logical but unclear about the consequences. Let’s take the unexpected shutdown event of a server far far away in another datacenter you would monitor.

You could end up with messages such as:

`Log Name: System<br></br>Source: EventLog<br></br><strong>Date: 24/08/2021 07:30:36</strong><br></br>Event ID: 6008<br></br>Task Category: None<br></br>Level: Error<br></br>Keywords: Classic<br></br>User: N/A<br></br>Computer: mycomputer.example.com<br></br>Description:<br></br>The previous system shutdown at <strong>09:30:12 on ‎24/‎08/‎2021 was</strong> unexpected.`

The timestamp of the event was perfectly recorded as a UTC time value which is converted by the application you are using on your own PC to look at the logs whereas the message that the previous shutdown was unexpected is a text message formatted by the machine in its own time zone.

To demonstrate this, you can execute the following powershell script. (You’ll hear a nice sound when your time zone changes).

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="powershell" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title=""># esent is just a source present on any windows pc
Write-EventLog -LogName Application -Source "ESENT" -EntryType Information -EventId 1234 "Hey I'm recording this event at $(Get-Date)"
$evt=Get-EventLog -LogName Application -InstanceId 1234  -Newest 1
$evt
# Look at the time field
$evt.Message
# Look at the time within the message (discard the error at the beginning)
$mytz=Get-TimeZone
(Get-TimeZone -ListAvailable)[-1] | Set-Timezone
# we're now in a different time zone
$evt
$evt.message
# Look at both times
# back to our time zone
Set-TimeZone $mytz
```

<figure class="wp-block-image size-large">[![](https://dimitri.janczak.net/wp-content/uploads/2021/08/Powershell-Time-Zone-EventLog-1-1024x515.png)](https://dimitri.janczak.net/wp-content/uploads/2021/08/Powershell-Time-Zone-EventLog-1.png)</figure>In the above example, the Time field is changed when you’re changing your time zone, whereas the message remains identical.