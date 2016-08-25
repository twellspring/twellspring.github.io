---
layout: post
title: Chatops Integration with Automation Tools
teaser: "Discovering ChatOps and a couple of hours of googling/reading lead me down the garden path of how to Integrate. I am using SaltStack for IT Automation ( sorry no Puppet/Chef here). SaltStack has served that role admirably. And now I want to move forward with ChatOps."
tags: hubot chatops saltstack
---

Discovering ChatOps and a couple of hours of googling/reading lead me down the garden path of how to Integrate. I am using [SaltStack](http://saltstack.com/) for IT Automation ( sorry no Puppet/Chef here). SaltStack has served that role admirably. And now I want to move forward with ChatOps. To do that of course ChatOps needs to talk to SaltStack, which means a bot. And the bot that seemed the best to me was [Hubot](https://hubot.github.com/) from those fine people at github that started this whole ChatOps thing.

Unfortunately, when I looked at how to get Hubot and Slack talking I let scope creep come in. I decided that a tool that already groked ChatOps and spoke both hubot-ese and slack-ese would be a good idea. This was a best intention for I wanted to make sure I was not contributing to integration hell with dozens of different integrations to our chat software Slack. There are already several different sources sending data into Slack. And ChatOps will be another one. And then there is SalesForce integration to slack. And I am sure many more to come. What If I could put a tool in place that would be the gateway between Slack and all of these tools. It would be a layer of abstraction that could be good to have.

And so I ended up trying two options: [StackStorm](https://stackstorm.com/) and [Rundeck](http://rundeck.org/). I liked both of these tools for different reasons, but I thought that the more dynamic event driven jobs scheduling model of StackStorm fit better. After all, ChatOps is also event driven so using StackStorm with its integrations to the major chat platforms seemed like a good solution. The StackStorm all-in-one installer got me up and running quickly, but with a few problems with the integration to SaltSack. Thanks to the very responsive ( and Slack based ) **community** group at StackStorm I was able to get it working.

And by working I mean I now have every salt command available inside of my Slack, including the ability to run arbitrary commands against any server … test, staging, PROD. Nope, not going to leave that in place. ChatOps needs some limits, so I set forth to figure out how to limit it to just what I wanted to expose. But hours later I was not really making progress on those limits. The integration layer between StackStorm and the Hubot bot that actually connected to Slack did not really seem to have the flexibility to do what I wanted. Or if it did it was going to take a lot of work to figure out and there was not a lot of information out there to help me.

Time to regroup and re-assess. The gateway ( StackStorm ) was making the job of integrating my SaltStack and Slack harder, and I was not convinced that my goal of using it as a single point of integration was really going to work. And so I jettisoned the scope creep and went back to a simple integration between SaltStack and Hubot.

Note: I really did like what StackStorm was trying to do and I am thinking that once I have some better experience with ChatOps under my belt I will be revisiting their solution as a dynamic job scheduler that might or might not integrate with the stand-alone hubot I am setting up now.
