---
layout: post
title: "Can Ops use Dev Techniques Part 2, Code Workflow"
teaser: "Another piece of the the development process the the code workflow.  We have bunch of developers writing code with the goal of providing new and hopefully useful features to our customers.  So we need a way to organize how all these separate pieces of code get merged into that finished product."
tags: devops, saltstack, gitflow
header:
    image_fullwidth: "mountains.jpg"
---

#### Piece 2: Code Workflow ####

Another piece of the the development "process" the the code workflow.  We have bunch of developers writing code with the goal of providing new and hopefully useful features to our customers.  So we need a way to organize how all these separate pieces of code get merged into that finished product.  This is code management.  After a couple of iterations, code management now typically uses a distributed code repository where multiple people can edit a piece of code and changes are added to the code base and then merged with other versions.  the most common tool for code management is [git](https://git-scm.com/). But it is a tool with much flexibility, so like the previous post a framework/methodology is needed.  With git these are usually called **branching strategies**.  And a popular one that our organization uses is **git flow**.

With git flow, one ore more developers will work on a branch created for a particular feature. These branches can be deployed to servers and tested.  Then once the feature is complete it is merged into the develop branch along with other features.  At some point, usually at the end of a sprint, a release branch is created to test all the new features together.  There will likely be changes needed to have them all working, which are made on the release branch.   Once it is fully functional and thoroughly tested, the release branch is merged in to production ( master ) and also to develop.

So can this development code workflow be used by operations too?  Well if the ops staff ( or the operations focused members of the DevOps team if you like that semantics better ) is doing everything by hand ( or with an loosely organized assortment of bash scripts), then the answer is no. So lets assume that Ops is using an automation tool to replace manually typed commands with "recipes" for creating servers, customizing the configuration and services available on those server and then deploying our applications to those servers.  Most of the automation tools ( like my current one [Saltstack](https://saltstack.com/) ) have these instructions written in files organized into a directory structure. Sounds a lot like "code", hence the term Infrastructure as Code ( IaC).

If Ops has automation code and more than one person working on it, then Ops is in the same code management boat as development.  So it seems reasonable that they manage it just like Dev.

Unlike Scrum where it really requires the full team to participate and make it work, a code workflow like git flow can be used by a few people or even a single person.  I found it pretty easy to incorporate into my daily practice and it is working well for well for my automation code. There are some ops specific issues you will have to work through, like how to securely put passwords and other sensitive data into the automation repository, but those are doable details.

Next post comes the last piece:  Deployment Workflow
