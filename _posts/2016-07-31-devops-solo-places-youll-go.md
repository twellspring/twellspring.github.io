---
layout: page
title: DevOps Solo. Oh, The places you'll go
teaser: "It was unexpected and a little ironic that the quote I found that best sums up my DevOps experience is from Dr. Seuss."
tags: devops
header:
    image_fullwidth: "mountains.jpg"
---

**&nbsp;&nbsp;&nbsp;&nbsp;You have brains in your head. You have feet in your shoes.<br>**
**&nbsp;&nbsp;&nbsp;&nbsp;You can steer yourself<br>**
**&nbsp;&nbsp;&nbsp;&nbsp;any direction you choose.<br>**
<br>
**&nbsp;&nbsp;&nbsp;&nbsp;You're on your own. And you know what you know. <br>**
**&nbsp;&nbsp;&nbsp;&nbsp;And YOU are the one who'll decide where to go.<br>**
<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Oh, The Places you'll go, Dr. Seuss

It was unexpected and a little ironic that the quote I found that best sums up my DevOps experience is from Dr. Seuss.

DevOps teams usually have people who specialize in Development or specialize in Operations. Thought there is overlap, at the end of the day (or sprint) the development focused team members do most of the code writing and the operations focused team members do most of the automating and infrastructure management.   While many DevOps theorists says there should not be this kind of specialization, it is usually what happens in practice and seems to work well enough.

I specialize in operations and have worked so far in what I call DevOps Solo:  a DevOps team that is small enough that there is only one operations specialist on the team.   As a DevOps Solo practitioner,  you are the person who designs the automation.  And implements it.  And maintains it.  Your time is divided between helping remove development roadblocks and moving the automation forward.  Or between improving the release cycle and implementing better security practices.   And you will have to rely on your own knowledge and experience to make this happen because **you're on your own**.

What are you talking about.  Your not on your own. There are blogs and podcasts and conferences where you can find like minded people and help.  Well, yes and no.  Yes there are good articles and speakers out there and they can be of help to the Solo practitioner, particularly when they are in the **technical how to** realm.
And No the articles are often not relevant to Solo.  Conference speakers and DevOps bloggers/podcasters are usually from larger DevOps organizations and frequently speak on issues that are only relevant to these larger teams. This is particularly true when discussing the cultural aspects of DevOps.  For example, migrating from traditional IT to DevOps really has no relevance to me as where I work never had traditional IT.  What does have relevance to me is how best to balance the tension between improving developers ability to ship code and implementing security procedures that slow down their ability to ship code.   And since I am a Solo, the time I have to then help the developers adjust their workflow to minimize the impact of those security procedures is limited.  So there is a more significant slowdown of development process than there would be in a large organization with more resources to dedicate.

So how does being a DevOps Solo look different from larger DevOps in practice?   Lets look at IT automation for an example.

All of the major IT automation tools ( Saltstack, Ansible, Chef, Puppet ) have community written modules you can use instead of writing your own automation code:

* SaltStack formulas
* Ansible playbooks
* Chef cookbooks
* Puppet modules

There is so much code available and I am thankful for the people who write them.  This kind of re-use is foundational to IT,  and I would really like to participate in this more.  But when **you're on your own** you often don't have that luxury.

When I need to implement new automation functionality I look at the available formulas first.  Most formulas are designed to work on as many different Linux versions as possible which means they are complex.  And they also are complex because they are designed to be customized based on pillar data making the flexible.  This means my decision to use or not to use them almost always comes down to complexity/functionality and vs simplicity/time.  And I almost always choose writing my own because:

* My initial needs are fairly simple
* I can get it working quickly
* The result is simple code
* I know exactly what it does and how to troubleshoot/extend it.

In a larger organization the well-thought out modules and well documented code provide a significant advantage if the team has just a little more resource available to understand and use these formulas as they are created, especially if that larger organization has a heterogeneous IT environment.  And  multiple Ops staff negates the advantage of a single person writing and knowing their code since every other member of the team will have to understand someone else's code regardless of who writes it.

This little more resource also applies to participating in the open source world.  I want to be able to contribute to SaltStack, both in enhancing formulas or contributing to the SaltStack code base. But a Solo has to implement the same basic set of DevOps tools and processes as a large team but with the resources of only one person.  Taking several hours a week to contribute to open source is just really hard.  In the end I squeeze it in infrequently, usually at the expense of my free ( read non work ) time.

I hope if I am ever out of the DevOps Solo world that I do have that little more resource so I can participate in these things. But for now I have to live with **You're on your own. And you know what you know. And YOU are the one who'll decide where to go.**
