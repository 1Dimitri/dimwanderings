---
id: 896
title: 'Architecture for the cloud vs. on-premises'
date: '2018-10-26T13:57:38+01:00'
author: Dimitri
layout: post
guid: 'https://dimitri.janczak.net/?p=896'
permalink: /2018/10/26/architecture-for-the-cloud-vs-on-premises/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:904;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2018/10/cloud-computing.png
categories:
    - architecture
tags:
    - cloud
    - design
    - IaaS
    - on-premises
    - PaaS
    - pattern
---

The “Architecture for the cloud vs. on-premises” theme has been too often polluted by sales, marketing and guts-opinions sometimes based on IT urban legends. In this post, we’ll try to take a picture of the landscape and see how the work habits should change… or remain the same.

# The Netflix myth

For example, if you ever discussed the cloud topic with top managers, there’s a good chance they tell you Netflix operates solely on AWS as their top success story as running all their infrastructure in the cloud. Most of them forget that Netflix operates their OpenConnect program: in order to ensure to their viewers an optimal experience, they propose to every ISP to host cache appliances of their own. So, they indeed put every user-interface related stuff in the cloud. But they designed their own hardware for what’s their core business: optimal viewer experience.

When you’re going to design your IT architecture, you should be careful to assess what is your core business and even think to go more specialized on some hardware part than you ever did.

# CAPEX vs. OPEX, aka costs

Often marveled at the promises of cost reduction, in particular because CAPEX suddenly become OPEX, firms may fail into the trap of huge recurring cloud bills:

- On-Premises, machines are powered 24/7 as electricity is treated in datacenters as a commodity, resulting in companies not assessing if the service should really run 24/7. You often have to go back to SLAs if time-window defined to assess if the service should be really running on nights and week-ends;
- In the cloud, every second of resource usage is paid, wherever in advance or on due term. So you have to know if your service should always be up or not and to weight this against the time the application takes to start and properly shutdown.

# Multi-vendor strategy

Like Windows vs. Unix in past Microsoft times, like Oracle vs. SQL Server or Postgres SQL back-end wars, each cloud vendor is likely to make you technical-solution dependent willingly or unwillingly. It can concern your application as an end product (e.g. your document store database) or even your CI/CD pipeline: how many of you can change their build process easily whatever the intended artifact (code, documentation but also virtual machine or even backup or — worse archiving storage format.

How to design then? Let’s review our options.

# Design methodology

What’s your application look like AND what should your application look like?

The pitfalls can be often found by answering the question:

- Do you have real figures and understanding in terms of system usage in its ecosystem?

Each word in this sentence has its own importance.

## Ecosystem:

- How autonomous is the application autonomous from any other company application. Ask yourself: 
    - how are users authorized and authenticated? You will perhaps discover dependencies OUTSIDE the application and in particular on infrastructure blocks the company never intended on one side or the other of the cloud / on-premises frontier.
    - where does the data comes from? Where is it output?
    - Checking the network ports on each component of your current solution can help you draw the effective flow of data in your company. This will also reveal the infrastructure dependencies.

## Real figures

- Do you have an up-to-date real time and historical diagram of the system including standard regular performance counters? This will help you refine the bits to be migrated/hosted in the cloud.
- CPU / RAM / Storage / SLA will give you insights about your IaaS, PaaS, SaaS strategy. If you have no figures, be ready for bills that vary over time.
- If you don’t have historical data, you may not know what the trend is and your solution may be totally coherent as of today but counterproductive in a few months. History every day or week will give you the trend but intraday statistics will help you confirm the SLA is OK and the measures you can take into the cloud to limit the resource consumption and your bill in IaaS scenarios.

## Understanding:

- Or Real figures can include “real feedback”: if all the users are unhappy with the current application, migrating as is a very bad idea. Redesigning may be your way. But selling the option to the stakeholders is often painful.

# Choice criterias

How to design your solution is a mix between

- the application pattern: how the components are split up or not
- Which elements are stateful (ie storing some kind of data in a broad sense) or stateless (you only have immutable pieces, code but also read-only data)?

# Application patterns

There are basically 5 patterns you’ll encounter:

- NTier
- Worker Queue Worker
- CQRS
- MicroServices
- Event-Driven

You should first note that there is no wrong or right architecture. Even for the same product the development history and the way the company has started in the Cloud, migrated from on-premises or maintain a hybrid strategy will output different results.

## N-Tier

The N-Tier pattern is the standard you get on premises. Typical example would be the Web Front-End, Application middle-tier server and Database back-end machines.

The easiest way is to migrate back and forth between cloud and premises is with IaaS by creating Virtual Machines. But remember to ask:

- is the current infrastructure correctly sized? What are the operation hours and timeslots?
- Can we hybrid only one part of the infrastructure? May be the DMBS back-end is too heavy in terms of data to be cost efficient in the cloud, but the application server would benefit for this. WHen hybriding the hybrid scenario be careful about RTO and RPO because you are multiplying the sources of SLA breaches.

## MicroServices

Microservices is the new trendy word. Basically you’ve cut your applications with as many slices of little functions or APIs you can.

Good thing is, it’s supposed to be stateless and can run on light services in the cloud such as Functions As a Service (you only host your function code and pay by execution times).

The bad thing comes from the composition of all these microservices. Since it is unlikely that you’re only computing the Fibonacci suite, you’ll end up to store and exchange data between those services. And here comes unnecessary cost transfert between those services, dependencies on API you do not necessarily maintain either in code (that’s a supplier) or in code technologies (you run your services on some technological and some other day the cloud vendor doesn’t support this anymore or your dual-vendor strategy is no more possible by some kind lock-in)

A good mind exercice is to assess if some of the services can be independently hosted on any cloud platform or in premises for the technological stack. And in that case, what are the additional costs your infer about data exchange

## CQRS

CQRS means Command and Query Responsibility Segregation. It is a way to distinguish streams based on read, write kind of operations. From a location perspective, it impacts the way you are billed since some vendors give free reads or free writes or bill you differently. Otherwise you can treat this as MicroServices as you still have to compose read &amp; write patterns as different services. Edge cases include hosting write and read patterns differently, such as having read on the cloud and write on premises to better isolate (from a security, responsiveness, immediate view of updated data…) point of view.

## Event-Driven Architecture

Event-Driven is not so much different from microservices in terms of composition madness. You’ve got producers creating data in a broad sense and you’ve got consumers which use that data. As it is often used in IoT scenarios, you may have more constraints on where to put some parts of your architecture, The IoT device may be a hardware device, so its location is likely to be ‘somewhere on earth linked to the Internet’, and there aren’t likely to be next to another in just a couple of datacenters. The frequency and latency of the data to be collected play a role where your put the data collection on the contrary.

If however all your producers and consumers belong to a private “guild”, depending on a cloud vendor is a not a pre-requisite and may introduce more points of failure on the long run than solve them

## Web Queue Worker

It is “just” a decoupled specialized Web Front ENd – Back-End application pattern. Treating it as a heavy compute, web front-end 2-Tier is usually enough.

# Stateful vs. Stateless

One other axis of the design should be how the various components are stateful or stateless. Basically, do they depend on some data context or variable.

Typically a IaaS infrastructure and its associated Virtual Machines are stateful (although booted from a somewhat stateless image), whereas a function (Amazon lambda function, Azure function) will be stateless.

Stateful vs. stateless can be obvious or not. A database, whatever its SQL or NoSQL flqvor, will be stateful and a Docker container, without any volume attached, once created is supposed to be stateless.

But the question may be a bit more tricker. In case you have a web server, where will you store the TLS Certificate which is somehow stateful (it has an expiry date, etc.), therefore should your web server be IaaS or PaaS oriented is also a discussion to have.

Once other aspect of the stateful debate is how often and quickly the data will change: from long-term read-only archiving for regulatory purposes to fast OLTP system, including large binaries rebuildable blobs, you choose various options:

- How large are your chunks of data?
- Is your data written once, read once, read many times, archived?
- Can be handled in parallel or should you use a FIFO access?
- Where is your data accessed from for creationg, updating, reading, backing up?

Examples of answer may then be.

- large binary streams can be files over HTTP or SMB, blobs, but medium binaries can be attributes or column in databases of different flavors
- Data written once could be a single file-backed database, replicated as such;
- Real-time updated data all over the world may still need a centrally managed RDBMS for its ACID properties.
- Use of Extract, Transfer and Load solution to replicate the data partially across regions of the globe

# Human factor

Many of the methodology points here can be found in books, on the cloud vendor site. However not many handle the human factor and in particular in big corporation, you may find a huge gap between what managers are told and what their employees will help you achieve.

- Convincing a on-premises company that part of the infrastructure will go to the cloud will raise barriers as people don’t in most cases their job evolve too much or, of course, disappear. This is valid also in the reverse way: a cloud-only company will not conceive that operating its own datacenter may reduce its bills because they will potentially see some specialized jobs positions to be created.
- The exercise requires a good map of your IT, past, present and future. People in companies find also difficult to recognize they do not everything perfectly by the book and that some IT system documentation is never written done but result from entreprise culture and customs.