---
layout: post
title: Chatops at 6 months. How has it changed development. Before
tags: hubot chatops slack
---

Yesterday one of my developers created a server environment to test a feature branch and had a problem: one of the 4 servers in the environment did not get built. A short investigation showed a typo on the ChatOps command was the cause. When I explained what happened and suggested building just that one server the developer replied "I will just terminate the environment and recreate it." Terminating and recreating servers is a normal/reasonable option for a situation like this, but for some reason that statement stuck in the back of my mind. A day later I realized why: it was said so casually. To the developer accomplishing what he said was easy. One Slack command would terminate the 3 servers and another command would build the 4 servers again. Why mess around with fixing a development environment that did not get built correctly the first time when you can create it again with 2 Slack commands.

The above experience was the catalyst for me to look a the impact ChatOps has had on development and the developer's thinking.  This is not going to discuss the technical aspects of making it happen.  That will have to be saved for another time.

My "discovery" of ChatOps was 6 months ago and my [first post](/blog/chatops-fomo/) was five months ago.  At that time, the developer was really not able to do that much.

#### A developer could ####
* ssh to a development server and change the code branch
* commit to a branch and a cron job would deploy the new branch

#### A developer could not ####
* Get a list of the current development servers
* Get details about a development server ( app installed, code branch, API server pointed at)
* Create a new development server
* Configure the new server as a particular role ( web server, app server, ... )
* Install/configure an application
* Setup the application to use a particular development version of an API.
* Terminate an un-needed development server

#### Server Testing Process ####

A web engineer would work on code locally and when they were ready to test it in a server environment follow a process like this:

* Figure out what servers and code branches they needed for testing.
* Ask the Ops team member what development servers were currently built/active.
* Ask around to see if any of the development web servers were currently unused.
  * If yes, ssh and change to the correct branch ( if they were command line savvy otherwise ask the Ops team member )
  * If no, either wait until it was available or ask Ops to create a new server.
* Ask around to see if any of the development api servers were currently unused **or** if all were in use was one of them on the needed branch of api.
  * If yes, ask Ops to point the web server at that api server. And in the case of unused ssh to server and change the branch ( if they were command line savvy otherwise ask Ops )
  * If no, either wait until it was available or ask Ops to create a new server.
* Proceed with testing.

 This process could take 30 minutes if all the planets aligned ( other developers responded promptly, servers were available, the Ops team member was available to repoint web servers ).  But I think on average it would take between several hours and two days.  And so a developer would have to plan ahead.  Back then I would frequently have developers say to me something like "I have a feature I am going to need to test next week.  Can you help me get the correct servers setup."

#### Thinking Process ####

Based on my interactions with the developers during that time, the typical developer though process was:

* Getting a new feature onto a server is hard
* I always need help from a busy Ops person
* I have to coordinate with 15 other developers
* Maybe I don't need to test this feature on a server environment.  An untested hotfix might be easier.
* It took so much work to get this environment setup as I want it so I am going to hold onto it even if I won't be doing any testing in it for a few weeks.

My [next post](/blog/chatops-at-6months-after/) will show how all of this has changed and the benefits it has provided to the developer and development workflow.
