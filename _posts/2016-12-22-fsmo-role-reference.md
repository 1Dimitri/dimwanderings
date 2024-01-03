---
id: 567
title: 'FSMO role reference'
date: '2016-12-22T17:15:26+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=567'
permalink: /2016/12/22/fsmo-role-reference/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"571";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2016/12/ADUC-Right-Click_Tasks-Menu.png
categories:
    - 'Active Directory'
tags:
    - ADDS
    - FSMO
    - principle
    - reference
---

As explained in [corresponding KB](https://support.microsoft.com/en-us/kb/197132), and as supposedly known by WIndows Active Directory administrators, here’s a quick guide, a FSMO role reference.

# Multi-Master Model vs. Single-Master Model

ACtive DIrectory is a multi-master model, which means changes are supposed to occur at any DC in the enterprise. However, it is not unlikely that since you can update data at any Domain Controller, sooner or later you will generate conflicts which may be begign… or catastrophic. For these potential catastrophic changes, MIcrosoft created the FSMO concept, which is basically, only one DC can change that value, that’s the SIngle Master words in FSMO = Flexible Single Master OPeration.

# Single-Master Model Roles

Currently in Windows there are five FSMO roles:

- Schema master (forest-wide)
- Domain naming master (forest-wide)
- RID master
- PDC emulator
- Infrastructure master

## Schema Master

The schema master FSMO role holder is the DC responsible for updating the directory schema you can find at LDAP://cn=schema,cn=configuration,dc=&lt;domain&gt;. You use it when you are introducing new Domain COntroller OSes in your forest, when you are installing Exchange, Lync/Skype for Business… or do [your own extensions](https://dimitri.janczak.net/2016/10/03/creating-a-custom-attribute-ad/).

## Domain Naming Master

The domain naming master FSMO role holder is the DC responsible for adding/removing domains in the forest and whose information is stored in LDAP://CN=Partitions, CN=Configuration, DC=&lt;domain&gt;. IN addition to the forest it is in, it also update references to domain external to the forest.

## RID Master

The RID master FSMO role holder is the single DC responsible for giving away blocks of RIDs to the other DCs in its domain.

Each SID in a domain is made of thre combination of the domain SID and a relative ID (RID) in the domain. Of course since the SID is the same for the domain, the RID must be unique so if you create objects on different DCs, the DCs should not give you twice the same RID. Therefore they request a set unique numbers to the RID Master in advance called RID Pool and allocate from it. THe RID Master should track the list so it doesn’t give twice the same pool of numbers.

Since it tracks RID numbers t is also responsible for removing an object from its domain and putting it in another domain during an object move.

## PDC Emulator

The PDC emulator is necessary to synchronize time in an enterprise. Windows includes the W32Time (Windows Time) time service that is required by the Kerberos authentication protocol. All Windows-based computers within an enterprise use a common time. The purpose of the time service is to ensure that the Windows Time service uses a hierarchical relationship that controls authority and does not permit loops to ensure appropriate common time usage.

The PDC emulator of a domain is authoritative for the domain. The PDC emulator at the root of the forest becomes authoritative for the enterprise, and should be configured to gather the time from an external source. All PDC FSMO role holders follow the hierarchy of domains in the selection of their in-bound time partner.

Haviong evolved from NT 4.0 times, the PDC emulator has the following functions:

- Time SYnchronization 
    - The PDC emulator is i is the authoritative time source for the domain. In the root domain of the forest, it becomes authoritative for all other domains.
- Password Account Management 
    - Password changes performed by other DCs are replicated first to the PDC emulator.
    - If you have an incorrect password attempt, this info is forwarded to the PDC emulator so the DCs can check they have the latest password hash. This avoids scenarii where you change passwords at one DC but try to connect immediately to another machine whose preferred DC is another one. The DC you’re connected to can then get the newest password hash without having to wait the replication to happen domain-wide
    - Account lockout is processed on the PDC emulator.

This part of the PDC emulator role becomes unnecessary when all workstations, member servers, and domain controllers that are running Windows NT 4.0 or earlier are all upgraded to Windows 2000. The PDC emulator still performs the other functions as described in a Windows 2000 environment.

## Infrastructure Master

Important in Windows 2000/2003 times, this is less and less the case as versions elvolve.

In those times, its role was to track objects from other domains as those objects had 3 components: a GUID, the SID (for security checks!), and the DN of the object as it is still an object in a LDAP Directory. When the Recycle Bin optional feature is enabled in WIndows 2008R2 or later, every DC is responsible to update its own link if object is moved, renamed, or deleted. In this case, there are no tasks associated with the Infrastructure FSMO role.