---
id: 36
title: 'Adopting a Cloud Platform: non-technical choices in an big environment'
date: '2014-09-19T10:02:03+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=36'
permalink: /2014/09/19/adopting-a-cloud-platform-non-technical-choices-in-an-big-environment/
layout_key:
    - ''
post_slider_check_key:
    - '0'
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:2:"43";s:11:"_thumb_type";s:5:"thumb";}'
image: /wp-content/uploads/2014/10/Victor-Hugo-meditation.jpg
categories:
    - thoughts
tags:
    - cloud
    - hyper-v
    - openstack
    - products
    - vmware
---

“Cloud” is the big marketing word today. If you work or have worked for a company whose size is more than a couple of dozens of employees you surely know that the non technical reasons will be a big part of the decision when it comes to choose a product to follow the trend.  
In particular for companies having their own data centers and still use their own IT, the challenges and road-blockers are huge:

- as usual, in a big company, everybody will resist: you are changing work habits, potentially teams and therefore balance of power, so expect resistance. I won’t deal with this aspect in this article because it is not cloud-specific
- some people, even IT, don’t understand that cloud is about automation and orchestration of IT process (build and maintenance of “environments”). Imagine, a couple of years ago, virtual machines was a revolution for some of them, they still try to figure out in which datacenter their program is running, and now you are adding several levels of complexity on top (or below, or around, as you see it) of the concept of hypervisor.
- interestingly, the hypervisor thing was hard to understand for a developer audience which, in general, is not interested in the gory details of system build and administration. The automation and orchestration thing is a more an issue for network people, system administrators, and such as you are going towards a “software-defined-and-centrally-managed” datacenter. We may see here the common old “protect my job” syndrom, In order to make this post fully appreciated by a full range of co-workers, here’s what you can expect: 
    - the network guy will surely not let his/her colleagues access his/her switches and routers;
    - as far as s/he concerned, the Unix/Linux system administrator won’t surely appreciate the idea of managing multiple servers at once if it’s not by launching the same script on dozen of machines from hosts not being from his/her collection;
    - the Windows system administrator won’t be annoyed as much by that since the Active Directory and GPOs have long introduced the concept of administrative boundaries bigger than a single server, but s/he has trouble with the command-line and that GUI interface they introduced with Windows 8.
    - it’s highly likely the corporation has developed its own tools,and that the tools from the shelf are used in every team but as little as interface or bus in mind. So you may expect that the various APIs rely on proprietary messages, verbose or non-documented XML (I’ll let choose you what you prefer). A not-so-subtle variation would be to use a CRM ticket system from which you cannot extract anything automatically nor properly formatted. Finally you’ll find that the “send mail to that guy” process is not bad at all.
- at the various managerial levels, the new company’s motto is “go cloud” has to be followed. Everything will be done to organize this, so meetings and demonstrations to show that the demand has been taken into account and at the end a product or provider, will be chosen with in most cases the help of people external to the company, i.e. consultants somewhat related to IT.

However, this may lead to dangerous surprises as in this very case, both IT people and managers are responsible for their own areas of “technology” and the choice of a platform must conciliate the following elements:

- still running the existing
- building the new infrastructure / Changing the current one to match the new design
- migrating if possible
- changing the reponsabilities
- choosing the product that will have as few as limitations as possible to help solving the above mentioned problems.

You will have therefore two main paths:

- take a niche product to avoid disturbing your IT organization as much as possible with the likeliness that at some point you are blamed for not fully implementing the “go cloud” motto especially if you are in the management chain;
- take a product that goes average in every technological domain, fulfilling the official motto, but creating resentments amongst many teams for a while. This can be worked out in particular if you have managers good at people or excellent people in a couple of teams whose knowledge and ability to work will overcome the resistance of others who are “just doing hours”

In this context, it is useful to note that several legal and marketing decisions have been made since the beginning of the year:

- examples of heavy-legally-constrained businesses have obtained the green-light from their authorities to host their data outside their own premises, such as banks in Belgium; therefore the excuse of “it’s illegal to go cloud in our business” is gone
- committees have been set-up to design various rules and labels, such as data protection in the EU. It will therefore add a gap between the solution providers who do comply with these regulations and the others, making harder to new software manufacturers to enter the “big companies” market. Examples of such regulations are the Article 39 of the [European Digital Rights](http://eur-lex.europa.eu/LexUriServ/LexUriServ.do?uri=OJ:L:2010:039:0005:0018:EN:PDF) directive.
- Traditional management-oriented summaries have been published about the market state: a couple of examples can be found easily [IaaS at Gartner’s](http://www.gartner.com/technology/reprints.do?id=1-1UM941C&ct=140529&st=sb), [x86-64 hypervisors at Gartner’s](http://www.gartner.com/technology/reprints.do?id=1-1WR6HLK&ct=140703&st=sb), [PaaS at Gartner’s](http://www.gartner.com/technology/reprints.do?id=1-1WR6HLK&ct=140703&st=sb), [Public Storage at Gartner’s](http://www.gartner.com/technology/reprints.do?id=1-1WWSLMM&ct=140709&st=sb)

If you compile all these studies, you’ll find that:

- current main players are Amazon Web Services and Microsoft; depending on how historically your IT is big and the ability of your market to deal with data not located on your premises, you want to choose one over the other solution. In particular, [regulations](http://ec.europa.eu/justice/data-protection/article-29/documentation/other-document/files/2014/20140402_microsoft.pdf) may play a big role if you absolutely need or are bound to them
- if you go niche, open-source based solutions are worthy to investigate, but expect to have people on-site taking time to create the glue
- .the applications you’re running are of importance : e.g. Support for Oracle is available for [Azure](http://msopentech.com/blog/2014/09/29/resources-oracle-database-weblogic-server-java-microsoft-azure), not for other platforms.