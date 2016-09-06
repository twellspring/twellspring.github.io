---
layout: post
title: "DevOps, Plums and Apricots or Pluots. Can Ops use Dev Techniques?"
teaser: "Is DevOps shoving plums and apricots into a bowl and pretending they are the same thing or is it breeding something new and giving it a new name like pluot?  When DevOps was coined in 2009, it was a label for a bottom up movement:  people in the IT trenches trying to figure out how to make things work better.  Agile programming had changed development and Ops was having trouble keeping up.  In the last 6 years the term has definitely gone corporate but retains its roots in the trenches.   So as a trench dweller I have to wrestle with the practicalities of DevOps every day.  Today I am wanting to see if the DevOps theory that Ops can use the same processes as Dev really works in practice."
tags: devops, saltstack,
header:
    image_fullwidth: "mountains.jpg"
---

Is DevOps shoving plums and apricots into a bowl and pretending they are the same thing or is it breeding something new and giving it a new name like pluot?  When DevOps was coined in 2009, it was a label for a bottom up movement:  people in the IT trenches trying to figure out how to make things work better.  Agile programming had changed development and Ops was having trouble keeping up.  In the last 6 years the term has definitely gone corporate but retains its roots in the trenches.   So as a trench dweller I have to wrestle with the practicalities of DevOps every day.  Today I am wanting to see if the DevOps theory that Ops can use the same processes as Dev really works in practice.

When I search Goole for "devops definition" the first result is [The Agile Admin](https://theagileadmin.com/what-is-devops/).  There are two definitions listed:

* DevOps is the practice of operations and development engineers participating together in the entire service lifecycle, from design through the development process to production support.
* DevOps is also characterized by operations staff making use many of the same techniques as developers for their systems work.

Can an operations staff managing servers, creating automation, supporting CI/CD solutions, and dealing with middle of the night outages use the same techniques as developers?  And if ops can, will they?  Why is it important that they do? Lets start by tearing development into some pieces to see if each piece can also be used by operations.  Today is piece one.

#### Piece 1: Development Methodologies/Frameworks ####
One piece of the development "process" is the methodology and/or framework used. At there essence these methods/frameworks are ways to organize how work gets done. Many organizations are using some form of Agile, frequently Scrum. For a team using Scrum, business requirements get turned into features that are broken down into small pieces.  The team prioritizes those pieces and selects a subset of those pieces they think can be completed within a defined period of time.  The selected pieces are divided amongst the staff assigned to the project who work on and hopefully complete them within that time period. At the end of the time period, the team reviews the results and makes plans for the next defined period of time. And the process repeats.

Can the above work for Ops? As an Ops person, I don't have a Product Manager or a Product Owner determining business requirements for the infrastructure. But I can look at the infrastructure as it exists today and make a list of what pieces are missing and what can be improved.  This list and feedback from management allows me to organize and prioritize my work.  And I do need to organize my work.  So there is nothing that prevents me AKA Ops from using something like Scrum.

Am I using Scrum for my Ops work?  At this point not really. I work daily with the developers in a DevOpsy way. but we are still trying to figure out the whole sprint thing after a period of rapid growth. Teams using a framework like Scrum have to go from framework to practice by trying things and seeing what works and what does not, then trying other things.  We tried a new tool for organizing our sprints that did not work well and Scrum suffered.  So we are changing that tool and re-booting our Scrum process.

I have been here before and know that this is a normal part of starting with a framework and ending up with functional practices.  It is messy, it takes time, but it eventually gets figured out.  And I think the end result is much better than starting with a more rigid methodology and trying to conform our practices to that methodology.  And so I think I will be back to using Scrum soon.

The next piece:  Code Workflow.  Stay tuned.
