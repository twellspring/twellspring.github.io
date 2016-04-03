---
layout: post
title: Hubot Carbon Copy a Channel
---

My pondering on how to clearly log Hubot initiated saltstack changes to a Slack channel without having a bunch of **help** and **pug me** in the channel lead to developing a method to carbon copy (cc).  A cc would allow developers communicate with Hubot via Direct Message (DM) but to have the results of any action command ( i.e. commands that initiate changes to a server) in the desired Slack channel.  The documentation did not cover this, and googling lead me to some outdated solutions that no longer work.  But a little persistence and some looking at the code lead me to the solution of using robot.messageRoom, which is now documented on the [Hubot scripting page ](https://github.com/github/hubot/blob/master/docs/scripting.md) ( or will be when [the pull request](https://github.com/github/hubot/pull/1161) is merged).

So with robot.messageRoom I can add a CC after the msg.reply lines in the examples from my previous post.

```
ccChannel = "some_channel"
if jsonBody.result != "failure"
  msg.reply "Successfully deployed #{appName} to #{serverName}"
  robot.messageRoom ccChannel, "Successfully deployed #{appName} to #{serverName}"
else
  msg.reply "Error deploying #{appName} to #{serverName}"
  robot.messageRoom ccChannel msg.reply "Error deploying #{appName} to #{serverName}"
```

Now when someone runs the deploy command via DM, in my channel that tracks staging changes I see a line

>Hubot BOT [6:22 PM]
>@someuser: Successfully deployed www to server xxx-xxx-xxx-1

which shows the change made and who made it.   That's just what we need, and I would have left it at this if I did not accidentally run a deploy command in the same channel as is defined for the cc.  And you guessed it ... duplicate messages.   That's not going to work.   And likewise I do not want to see duplicate messages when I am testing new commands locally via the Hubot command line.   So we now need to turn this into a function that has the logic necessary to suppress duplicates and can be called from multiple locations.  Creating a local function is my first try and I got that working without too much frustration and head-banging.  I decided to stick with "room" in my code as that is the generic term Hubot uses which in Slack's case maps to a channel.


```
module.exports = (robot) ->

  ccRoom = "some_channel"

  cc = (msg, ccRoom, ccText) ->
    room = msg.envelope.room
    user = msg.envelope.user.name

    return if ccRoom == room
    return if user == "Shell"

    robot.messageRoom ccRoom "#{ccText}"
```

A couple of CoffeeScript notes here.   The CoffeeScript **return** call will exit the function without processing any more lines.  And the pattern "command if test" means those means those lines only get executed if the condition statement evaluates to true.  I like what I did above as the code is cleaner and more readable than a typical compound if-then-else

```
if ccRoom == room
  return
else
  if user == "Shell"
    return
  else
    robot.messageRoom ccRoom "#{ccText}"
```

or using an **or** in the if statement.

```
if ccRoom == room or user == "Shell"
  return
else
  robot.messageRoom ccRoom "#{ccText}"
```

Notes done ... back to the originally scheduled subject.   Using this new function requires replacing each robot.messageRoom line with a cc line.

```
  cc(msg, ccRoom, "Successfully deployed #{appName} to #{serverName}")
```

While this works, I already have a couple of CoffeeScript files and this definition needs to be at the top of each one that needs to do a cc.  That is only one file for now, but I am sure there will be many more in the near future.  So using the local function is not a long term viable solution.  I need to be able to define this globally, and that ccRoom variable also needs to be defined globally.  One step forward, two steps back.
