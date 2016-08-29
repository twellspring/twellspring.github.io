---
layout: page
title: Using Hubot Middleware to Restrict Rooms
teaser: "Today's post is a small diversion from the security topic I planned because something came up that I wanted to address first ... where hubot could be accessed from"
tags: hubot chatops slack security
header:
    image_fullwidth: "mountains.jpg"
---

Today's post is a small diversion from the security topic I planned because something came up that I wanted to address first ... where hubot could be accessed from.   Slack's Hubot integration creates Hubot as a robot similar to slackbot, except that hubot has to be invited to a channel where slackbot is already listening in every channel.   Since Hubot is being used for ChatOps, I want to limit where server commands can be executed.  Limiting what channel hubot is in does not work as anyone can invite hubot to a new channel.    So the limits need to be within hubot.   This could be done on a per command basis, but that complexity and granularity is not needed at this time. So for now it will be a global limit on all hubot commands to certain channels.

[Hubot Middleware](https://github.com/github/hubot/blob/master/docs/scripting.md#execution-process-and-api) is the tool for this job.  The room limiting logic will be a white list.  Any command in a room not on the white list will be silently dropped.

Time for a new file `middleware.coffee`

```
# Description:
#   Middleware Commands
#
# Dependencies:
#
# Commands:
#
# Notes:
#

module.exports = (robot) ->

  robot.receiveMiddleware (context, next, done) ->
    room = context.response.message.room
    next() if room in ['staging', 'testing'  ]
    done()
```

The receiveMiddleware acts after the listener ( robot.respond, ... ) matches but before the body of the listener function executes.  It gets the room from the context object and compares it to a list.  If there is a match the `next()` sends the request on to be processed to normally.   If it does not match `done()` silently stops this request and nothing more happens.

A quick test in my local development environment shows a problem.   No commands now work.   Adding a logging line after the room value is set ( `console.log "room = #{room}"` ) shows the issue.   When using locally, the room is **Shell**, same as the user name.    I could put Shell in the list, but putting in the user name value rather than a fixed **Shell** will also allow Slack DM to work as Slack Direct Messages also have a room name equal to the user name.    Here is the improved middleware.

```
  # Limit responses to the listed rooms, Shell and Slack DM
  robot.receiveMiddleware (context, next, done) ->
    room = context.response.message.room
    user = context.response.message.user.name
    next() if room in ['staging', 'testing', user ]
    done()
```

Done and done.  Now even if a user invites Hubot somewhere he should not be, Hubot will not respond.
