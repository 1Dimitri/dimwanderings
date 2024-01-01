---
id: 329
title: 'URI differences: URN, URL&#8230;'
date: '2015-08-04T19:31:12+01:00'
author: Dimitri
layout: post
guid: 'http://dimitri.janczak.net/?p=329'
permalink: /2015/08/04/uri-differences-urn-url/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";s:3:"328";s:11:"_thumb_type";s:5:"thumb";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
image: /wp-content/uploads/2015/08/URN-URL-URI.png
categories:
    - terminology
tags:
    - 'uniform resource'
    - URI
    - URL
    - URN
---

You are probably using those concepts without knowing: today let’s speak about URI differences.

## URLs: Uniform Resource Locator

Most of us know what is an URL even they are very basic computer users: that’s a way to locate a file somewhere on the network. Everybody will speak about [http://](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) and [https://](https://en.wikipedia.org/wiki/HTTPS) prefixed URLs. The older you are, the more likely you’ll also list [ftp://](https://en.wikipedia.org/wiki/File_Transfer_Protocol), [gopher://](https://en.wikipedia.org/wiki/Gopher_%28protocol%29) and and alike.

What most people don’t explicit tell you is that in fact it is a combination of a scheme and a location that should be totally resolvable to get an existing resource from the network.

However explanations are less clear when you ask people about URIs and URNs.

## URIs: Uniform Resource Identifier

URIs are easy: there are the combination of URLs and URNs. Nothing more to add here.

## URNs: Uniform Resource Name

URNs are more fuzzy depending on the source you are referring to. If you are strict, URNs are everything that start with urn:. You just put something after it in order for you to have a unique name for giving identity to your resource. For example, when you must name your application in a [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) world or your XML schema to the outside world you can create whatever unique name you like after urn:.

General practice nonetheless allows URNs to start with something else than urn, such as tel: for a telephone number. You can also invent your own, like if you were a start-up creator, should you invent a new way to communicate. You then end up with something:somethingelse.

Finally, you may want to note that when an URI is requested people may choose between URNs and non-existing URLs (such as http://myapp.local/) which add to the confusion of these 3 concepts.

The drawing accompanying this article summarizes those differences in a traditional set design:

[![URN-URL-URI](http://dimitri.janczak.net/wp-content/uploads/2015/08/URN-URL-URI.png)](http://dimitri.janczak.net/wp-content/uploads/2015/08/URN-URL-URI.png)