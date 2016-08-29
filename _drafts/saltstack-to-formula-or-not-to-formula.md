---
layout: page
title: Saltstack: to formula or not to formula
tags: saltstack
---

Starting out with Saltstack I was, not surprisingly. focused on learning the basics like how states worked and then how to use a pillar to pass data to my states.  Those first states took a lot of work because I was constantly reading and re-reading documentation and then trying things until they worked as I needed.  During this proces I discovered [SaltStack Formulas](https://github.com/saltstack-formulas).  At first this was a great find ... a whole bunch of saltstack states that I could use either as states or as examples of how to develop my own states.  Then I read some of them.

Being a salt newbie, the states I read were a complex and hard for me to understand.  I could understand enough though to realize that a large part of the complexity was making the recipe platform agnostic ( i.e. they would work on Ubuntu or RedHat or .... ).   But since we only used Ubuntu, that agnosticism did not add any immediate value.  I could see people had put a lot of work into these formulas and would certainly have though of things I would not if I wrote my own states. But was the effort required to figure out how to use these complex formulas worth it, or should I write my own simpler states.

The first case was nginx.   I found a well developed [Nginx formula](https://github.com/saltstack-formulas/nginx-formula) with thousands of lines of sls code.   And I wanted to understand that code before I used it in my project.  Maybe I did not need to understand every line, but I needed to have a reasonable idea of the how's and why's.
