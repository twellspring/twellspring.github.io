---
layout: page
title: Becoming Hubot or How I make Hubot look really smart
teaser: "When my teammates talk to me about problems with Hubot I learn about what is and is not working well, which often leads to code updates to make Hubot easier to user. Recently my conversations are indicating teammates are having difficulty with getting the hubot syntax correct."
tags: hubot chatops slack
header:
    image_fullwidth: "mountains.jpg"
---

When my teammates talk to me about problems with Hubot I learn about what is and is not working well, which often leads to code updates to make Hubot easier to user. Recently my conversations are indicating teammates are having difficulty with getting the hubot syntax correct.  They know about the help command, but they are not able to interpret it.   This is particularly true for those with little command line experience.

So I went to look for a way of seeing what mistakes they are making.  Since these mistakes are not matching a Hubot command there are no log entries for them.  So how can I see these?   My solution was to become Hubot.  Well maybe not actually, but to become enough of Hubot to see what users are doing when they are not getting the commands right.  A little digging I discovered someone had already thought about this and created the **robot.catchAll** command.   I set about to use this to take any commands that did not match and send them to me so I can see the mistakes.

```
  robot.catchAll (msg) ->
    hubotAdmin = process.env.HUBOT_ADMIN
    robot.messageRoom hubotAdmin, "#{msg.message.user.name}: #{msg.message.text}"
    msg.finish()
```

My testing of this in local hubot looked good, but I was a little suspicious so I put it into PROD at a low use time so I could try it out without a lot of volume.  Good thing I did.   I discovered that the above would match on any message sent to a room that Hubot was listening in, even if it did not start with the hubot name/alias ( usually ! ).   Hubot is listening in several busy channels, so I would get a **lot** of messages that were not failed hubot commands.  Quick revert out of PROD.

Take two.  I need a filter on this command to only match if the hubot prefix is there. Thanks [wingrunr21](https://gist.github.com/wingrunr21) for [catchall.coffee](https://gist.github.com/wingrunr21/7118744) which showed me the way to do this.  Here is what I now have in place.

```
  robot.catchAll (msg) ->
    hubotAdmin = process.env.HUBOT_ADMIN
    regex = new RegExp "^(?:#{robot.alias}|#{robot.name}) (.*)", "i"
    matches = msg.message.text.match(regex)
    if matches != null && matches.length > 1
      robot.messageRoom hubotAdmin, "NOMATCH #{msg.message.user.name}: #{msg.message.text}"
      robot.logger.info "NOMATCH #{msg.message.user.name}: #{msg.message.text}"
      msg.finish()
```

This sends me a copy of only failed commands and logs the same.  And NOMATCH in front allows me to search/filter in my logs or in Slack to get a list of just these invalid commands.  Once that is in place for a month I will be able to analyze the data and see trends.

But that does not help my users now.  How can I put this data to use today?  Getting "Hubot support" until now has been talk to the admin ( me ) via slack.  They ask me a question and I respond to them.  But lets take support a stop further.  What if I become Hubot so that Hubot can respond to the user when they are having troubles with syntax.   I could send as Hubot something like this:

```
It looks like you have some extra spaces between the words in the
command you are trying to run.  Try removing those.
```

That would give a much better end user experience and make Hubot sound really smart. So I created an **echouser** command only available to admins.  Since this is only for admins I have omitted help for this command so users do not see it.

```
robot.respond /echouser ([\w-]+) (.*)$/i, (msg) ->
  if robot.auth.isAdmin(msg.envelope.user)
    room = msg.match[2]
    message = msg.match[3]
    robot.messageRoom room, message
    msg.reply "Message sent"
```

Now when a user makes mistakes in syntax I will see them and I can respond with a helpful suggestion ... and occasionally a snarky response as per Hubot best practices.

```
All of your syntax mistakes are causing overload in my OCD algorithms.  The help command is there for a reason.  I suggest you use it.
```
