---
id: 256
title: 'Reading Review: Masterminds of Programming: Conversations with the Creators of Major Programming Languages'
date: '2015-07-12T14:24:52+01:00'
author: Dimitri
layout: article
guid: 'http://dimitri.janczak.net/?p=256'
permalink: /2015/07/12/reading-review-masterminds-of-programming-conversations-with-the-creators-of-major-programming-languages/
tc-thumb-fld:
    - 'a:2:{s:9:"_thumb_id";i:263;s:11:"_thumb_type";s:10:"attachment";}'
layout_key:
    - ''
post_slider_check_key:
    - '0'
categories:
    - readings
tags:
    - programming
    - 'project management'
---

[![Masterminds of programming](http://dimitri.janczak.net/wp-content/uploads/2015/07/Masterminds_of_programming.jpg)](http://dimitri.janczak.net/wp-content/uploads/2015/07/Masterminds_of_programming.jpg)Although a bit ancient (2009), I had the opportunity to get an used copy of “Masterminds of Programming: Conversations with the Creators of Major Programming Languages” (ISBN 978-0596515171). This is a set of interviews with people who designed and/or implemented languages in a broad sense: programming language but also UML, etc.

Depending on the interviewee, the interview is more technical or more general, more serious or enjoyable but you’ll learn always something.

On the amusing side, you’ll learn that atomicity has been introduced in SQL as a solution to the Halloween problem which popped up just before this long week-end when the authors realized that a script to calculate pay changes was raising the amount twice because rows appeared twice or more as soon as their position changed in the index.

On the human side, you’ll notice that the Objective-C creators find the C++ too bloated for its purpose whereas the two projects started with the same idea to circumvent C limitations, that the UML creator finds the current separation between architects, developers, testers project managers as counterproductive since you never know where your project is.

You’ll also see the difference between:

- people who designed a piece of software that was just designed to solve their issue and was finally widely adopted, like AWK or Lua, who seem to be genuine and friendly
- people responsible for cathedral languages like Java and C++ for which their broad usage makes their creators very serious and subject to amusement from the first category of people
- leader and manager, i.e. long-term vision vs. day-to-day task handling

Interestingly, that distinction crosses the boundaries of mathematicians vs. non-mathematicians.

You can also pick up a few ideas for your IT day-to-day practice, such as

- The right-balance between teams and single programmer work seem to be around 6-8 people because of the communication efforts or lack of it.
- UML, according to its creators has too many constructs, you should restrict yourself to 20% of its grammar to be performing. If you want to avoid such bloated definitions, never accept to have normalizing committees because of the participant will try to add its block used solely by himself.
- Many interviewee find the IT industry too childish and fashion-aware as we tend to want to re-invent from scratch every 2 or 3 years, but in the mean time
- A good IT guy is the one who shows creativity, whatever the task, according to most of them
- People don’t read documentation, so don’t bother writing it except to get a general idea of what your software or code does
- The maintenance of existing software is an issue because most software weren’t designed with the time they will stay in production in mind and because freshmen are trained on new technologies, not legacy or even current ones, which do not interested them anyway.