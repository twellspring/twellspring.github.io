---
layout: post
title: Hubot and Slack Security concerns
---

anyone can add hubot to a roop
anyone can run any command

First out was sub-routine at the top of each script that limited PROD access to only admins ( defined in env).  Staging servers can be created/delted/updated by anyone in the staging room ( sub-routine checks room and says sorry charlie if it is not staging)

Next will be middleware to handle the above

eventually there will be roles to give better control of
