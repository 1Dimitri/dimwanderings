---
id: 752
title: 'Windows Server OS unattended installation before DevOps'
date: '2017-08-06T08:00:07+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=752'
permalink: /2017/08/06/windows-server-os-unattended-installation-before-devops/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"761";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2017/08/Bart-Network-Boot-DIsk-French-Protocol-Selection.png
categories:
    - 'computer history'
    - Python
    - 'Windows OS'
tags:
    - python
    - 'Windows PE'
---

With Azure and the Container hype, Microsoft is pushing developers and system administrators to automate everything. The graphical Windows platform has become easy to automate with Powershell and the latest versions of the OS. But, already faced with huge server pools decades ago, some of us already had automation workarounds in place when Microsoft didn’t support this very well officially. By popular request, here’s a personal story of what Windows Server OS unattended installation before DevOps looked like.

# Windows NT 3.51 / early NT 4

Urban legend says Bill Gates decided to link the NT Native API with the graphical Win32 subsystem. At that time, 1997, I worked for a DYI retail chain whose solution to automated installation was simple. Since all shops had roughly the same needs, all servers where identical and were prepared from a ‘shop zero’ template which was then ghosted. In fact, not really ghosted, but merely whose RAID 1 disks were regularly put in new servers to ‘rebuild’ the mirror. Neat, efficient, but the limits of the system started to appear in early 2001 when we got our hands on the first Windows 2000, but [newsid](https://docs.microsoft.com/en-us/sysinternals/downloads/newsid) was our friend at that time.

# Windows NT 4 reloaded

I had the opportunity to start developing unattended installations when I changed work in 2001. In my new job, I had to prepare servers for a variety of configurations and ghosting was no way. The manual procedure was boring, not human-fool proof and you had to perform the installation most of the time in noisy server rooms. There was a huge incentive to develop something which would leave you with something error proof and quiet once you bootstrapped it.

The main issues were: booting under DOS, installation of third party drivers for servers named Compaq in that era, and multiple pop-up windows which were not fully automatable in spite of the [SendKeys](https://msdn.microsoft.com/en-us/library/microsoft.visualbasic.compatibility.vb6.support.sendkeys(v=vs.110).aspx) trick which is probably the reason of the success of Visual Basic ([Visual Basic](https://en.wikipedia.org/wiki/Visual_Basic), not[ Visual Basic.Net](https://en.wikipedia.org/wiki/Visual_Basic_.NET), young readers). In addition my lack of knowledge and lack of access to the Technet CDs (once again, young reader, there was a non-web version of technet in the past).

The prototype was never fully deveployed for all servers as some hardware drivers had restrictions or installation files which would not fit the full unattended model.

# Windows 2003, here comes the fun

Windows 2003 was more interesting. Probably as a side-effect of the transition of the end-user market from WIndows 9x to XP, many automated solutions Microsoft developed for the OEM vendors found their way into the server area. It was a time of:

- Bart Lagerweij’s Network Boot disk to start from DOS (hey, I found a link with [my contributions -PCISCAN patch and n1000 driver- to that project](http://severinterrier.free.fr/Boot/NetBoot.htm))
- Soon enough, finding a suitable replacement to perform mostly diskpart-align-related actions on NTFS disks and bootstraping the 32-bit setup instead of the 16-bit version: it was the beginning of Windows PE and of its [Bart PE](https://en.wikipedia.org/wiki/BartPE) side kick. At that time, very young reader, Microsoft was not very Open Source friendly, and if you were not an OEM Supplier, you could not get a copy of Windows PE.
- In the first years, installing an additional Windows copy side-by-side with the production one; driver updates were not BSOD-proof and editing files and registry was much more comfortable for a full-fledged Windows than from Windows PE. In order to achieve this, I used a combination of [BOOT.INI entries](https://support.microsoft.com/en-us/help/323427/how-to-manually-edit-the-boot-ini-file-in-a-windows-server-2003-enviro) and the entries Profiles, ProgramFilesDr and COmmonProgramFilesDIr in sections GuiUnattended and Unattended in [WINNT.SIF](http://www.msfn.org/board/topic/43736-changing-programfilesdir-directory/)
- The fight for having a syntactically correct version of the [TXTSETUP.SIF](http://www.msfn.org/board/topic/102617-txtsetupsif/)/[TXTSETUP.OEM](https://msdn.microsoft.com/en-us/library/windows/hardware/ff544738(v=vs.85).aspx) and .INF files by carefully distinguishing text modes and GUI modes. Your main concern when receiving new hardware was to know if the storage and network card manufacturer had proper drivers for these two phases and if no mistakes were made such as incorrect relative paths in the .INF file. Fortunately, it was a time of peace and romance and no .CAT file was enforced to check the driver’s integrity so amending text files was easy
- The glorious days of the I[nstallshield wizard setup.iss](http://helpnet.installshield.com/installshield22helplib/helplibrary/CreatetheResponseFile.htm) file when msi files were not the majority of third party products
- Finding a scripting language that could glue all of this together. I choose Python for three reasons: 
    - language forgiving library: extracting the 10th character of a 9-character length string wouldn’t raise an error, the map/zip function were very powerful on string lists where I stored my installation paramerters and not so many language had list comprehension at that time.
    - [Win32 extensions ](https://sourceforge.net/projects/pywin32/)by Mark Hammond which lead to have py2exe projects.
    - Code was easy to write and read even if you didn’t touch it for months

It was so efficient that it allowed us to:

- install hundred of servers
- re-install them for other purposes when projects were done
- fulfill audit requirements ( a versioned parameter text file and a log file allowed to guarantee the tracking of what’s installed)
- in a consistent fashion

# Windows 2008 (R2): industrialization times

I was ready to use the same kind of magic on the next OS, but I discovered what was at that time called BDD. Like many, I soon realized that with the alignment of kernels between the 64-bit client OS and the server OS, there should be no reason that this tool could not be used for Windows 2008/R2. Teams at Microsoft probably had the same idea since the BDD product was soon renamed MDT (Microsoft Deployment Toolkit).

I loved my Python-based solution, and I had hard time fighting with the cumbersome VBScript language, but I made the decision to switch over the Microsoft solution for support reasons, as we had a Premier contract and I highly doubted that any PFE would debug my Python code.

Changes were made though to some aspects of the LTI process:

- Many scripts were added/updated for example to support DNS Suffix search order, install packages not based on geographical location, but areas of team responsibilities…
- In order to fulfill audit requirements and to quickly generate new servers, the .ini/database based parameter process was replaced by a versionedXML file for each server containing all parameters. At that time XML was a sound choice and JSON was not even supported by the library routines provided by Microsoft.
- At the same, Microsoft wildly distributed Windows PE which altogether with WIndows Deployment Services solved our 64-bit only server head-ache.

Today, the process is still used to create new servers for legacy applications. The main problem is to update the Windows 2008 R2 WIM as from the official WIM to an usable server, you have to:

- replace IE 8 by IE 11 not forgetting the prerequisites;
- update the build with the Service Pack 2? err, sorry, marketing guys, I mean the [convenience rollup](https://support.microsoft.com/en-us/help/3125574/convenience-rollup-update-for-windows-7-sp1-and-windows-server-2008-r2), which itself kinda replaced the [enterprise rollup](https://support.microsoft.com/en-us/help/2775511/an-enterprise-hotfix-rollup-is-available-for-windows-7-sp1-and-windows);
- have a load of additional task sequences to install the latest .NET framework, its patches, and the Windows Management Framework as they cannot be officially slipstreamed.

# Windows 2012(R2)/2016: everybody easily does it, doesn’it?

Those two are finally very simple in comparison to the previous generations. As the rumor says the first Azure servers were running a modified Windows 2008 version, and as the core edition had to be supported anyhow, Microsoft had time to refine its unattended installation process.

Internally the process I use has moved away from XML files to describe the machines to install and now use Powershell hashtables definition which are boxed/unboxed to/from MDT/SCCM environment variables. It allowed to still maintain the version control of the parameters without to backup an entire database and also to create a business layer, where we can easily validate some scenarios where some fields are required if some values are present for example. A possible evolution would be to port those tests under Pester.

IMHO The main pain points are though:

- an emphasis on Powershell whose cmdlets vary from version to version but also from OS generation to OS generation. (Try Get-SMBShare on Windows 20078R2 even with the latest Powershell version installed)
- An inability to have an efficient, consistent and proper update model which works for live and offline systems. Oh, dear core team, why on earth has the monthly rollup changed so many times, why should I patch every index image in a WIM when I apply a package, why do I get unattended xml error on some packages if I’m not aware of the servicing stack packages order?
- In addition, the lack of distribution of up-to-date OS images on the Volume License Center add a burden to the system administrators.

Let’s hope however that having to maintain their Azure infrastructure, the MS IT has the same issues as me and will finally find a solution in the next Windows 10 ‘Don’t call me WIndows 11 because the mrketing says Windows 10 is the last one’ update.