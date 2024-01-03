---
id: 981
title: 'Microservices architecture best and worst practices'
date: '2019-10-14T12:29:32+01:00'
author: Dimitri
layout: article
guid: 'https://dimitri.janczak.net/?p=981'
permalink: /2019/10/14/microservices-architecture-best-and-worst-practices/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:984;s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2019/10/Avoiding-Microservice-Megadisasters-Jimmy-Bogard-YouTube.png
categories:
    - architecture
tags:
    - 'best practices'
    - microservices
---

Regularly, the IT world discovers the alpha and omega patterns that should solve all the software or hardware problems. Recently, Microservices has been a trend and of course it is time to list the microservices architecture best and worst practices.

In a [youtube video “Avoiding Microservice megadisasters”](https://youtu.be/gfh-VCTwMw8), Jimmy Bogard outlines his experience with refactoring a full micro-services line of business application whose original design didn’t work. According to him:

- The popular belief about not replicating data doesn’t allow proper response. It’s OK to replicate data, as long as it is read-only;
- WebAPI for everything leads to API hell: too many calls will go far beyond any acceptable SLA because of messaging and network latency. E.g. 200 Web APIs with very low latency WCF 150ms service can never make it even with a 99.9% uptime.
- More generally, never assume, like a developer on his own machone, that: 
    - the network is reliable
    - the latency is zero
    - the bandwidth is infinite
    - the topology doesn’t change
    - the network is the same at every location
    - the network is secure
    - there is only one team for administering
- When separating into services, [standard marketing slides such as the one by Microsoft](https://azure.microsoft.com/en-us/blog/microservices-an-application-revolution-powered-by-the-cloud/) do lead to issues as they are not correctly understood. Breaking the services into layers and services for each layer do not generally break to any business unit or organization. E.g. you do not have a “Usage Analytics” responsible unit in the following picture:![Tr aditional MS microservice split](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/f05a4c2f-753a-408a-9732-d76d06c70604.png)Unfortunately, splitting is often rewarding as you need more people and managers are often assessed by the number of people managed.
- You must remind yourself what is a context: e.g. if a customer doesn’t mean the same thing for two teams, you need different isolated contexts for them and define communications between those. This is a bounded context, and that’s [The Domain Driven Design book](https://dddcommunity.org/book/evans_2003/)‘s main lesson which is at the end of book.
- Do not hesitate to reverse service dependencies. If your service depends on 100+ API, their SLA, and has no data, it’s maybe time to make its own service as: 
    - it’s owning the relevance of the data it presents
    - it’s owning its own SLA
    - it should in fact owning its own data
- and have the other services call it to populate that data to solve the last item issue and remove any temporal/SLA issue.
- Then you can isolate the external access to your service which has no longer any dependencies and the internal works needed to get your read-only data.
- In a real-enterprise world, the way you manage to get your hands on data (database triggers, files, infrequent -as in not real-time- API calls etc.) doesn’t matter
- A rule-of-thumb to good design, is that your service has the right to call services one-level deep but not more. If the service you’re calling needs to call a service, you must change your boundaries.
- Additionally, there are thoughts about how (broken) micro-services at the end mimic the (broken) overall organization of the enterprise, following [Conway’s law](https://en.wikipedia.org/wiki/Conway%27s_law).

It’s worth noting that the books referred to are:

- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)
- [Building Microservices](http://shop.oreilly.com/product/0636920033158.do)
- [SOA Patterns](https://www.manning.com/books/soa-patterns)