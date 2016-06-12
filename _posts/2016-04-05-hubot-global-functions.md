---
layout: post
title: Hubot Global Functions
tags: hubot chatops slack
---

Researching global functions in Hubot lead me down two interesting paths: Classes and Middleware.  Middleware is pretty well documented on the [Hubot Scripting page](https://github.com/github/hubot/blob/master/docs/scripting.md).  Classes on the other hand not so much.   So for learning about those ... you guessed it ... go to the code.  And with that I see that while Middleware has other uses, the Class is what I will use to create what I am calling a **global function** (in javascript terminology this is a class with a method).   In another .coffee file in the scripts directory, create a Class and then a method in that class for the CC script. I decided to create a class with my company name as the class.  I will add other global functions under this same class so they are all grouped under my company name.  This methodology is likely to change as I learn more, but its working for now.  Here is what it looks like.

```
module.exports = (robot) ->

  class myCompany

    cc: (msg, ccRoom, ccText) ->
      room = msg.envelope.room
      user = msg.envelope.user.name

      return if ccRoom == room
      return if user == "Shell"

      robot.messageRoom ccRoom "#{ccText}"
```

Notice I had to change the = after the local function name into a colin for method name.  Other than that the function could be moved as is.   Now to call it.   In my code replace the cc line with

```
  robot.myCompany.cc(msg, ccRoom, "Successfully deployed #{appName} to #{serverName}")
```

That makes pretty quick work out of the first half.   Now what to do with that global variable.  As I thought about this I realized one global variable would not solve the problem.  Where a CC goes will depend on the input.   In my case I wanted to be able to cc a different room based on the environment of the server that a command is being run on ( test, stage, prod).   This will require 3 pieces

* a method to get an environment out of a server name
* a map of strings to room name
* a call to the map to convert the server environment to the room name

Here are the three pieces in code.

```
server_split = server_name.split "-"
server_env = server_split[3]

slackRoomMap =
  test: process.env.HUBOT_SLACK_TEST_ROOM
  stage: process.env.HUBOT_SLACK_STAGING_ROOM
  prod: process.env.HUBOT_SLACK_PROD_ROOM

ccRoom = slackRoomMap[ccString] ? ccString

```

Using the split method to get the environment works because my server names are well formed and the 4th section is always an environment.   Since the split function array starts at 0, server_split[3] is the fourth object in the array.  The slackRoom map is a basic CoffeeScript object literal. And replyRoom is set using the bracket notation to access the appropriate value in the object literal.  ? is an existential operator which returns the value after the ? if the value before does not exist or is null.  In English, this means if there is no match in the slackRoomMap to treat the passed ccSting as ccRoom.  So I can now pass either an environment name or a room name and the CC will go to the appropriate room in either case.

My revised global function now looks like this.   Note the change in the ccRoom in the method definition to ccString.

```
module.exports = (robot) ->

  class myCompany

    cc: (msg, ccString, ccText) ->
      room = msg.envelope.room
      user = msg.envelope.user.name

      slackRoomMap =
        test: process.env.HUBOT_SLACK_TEST_ROOM
        stage: process.env.HUBOT_SLACK_STAGING_ROOM
        prod: process.env.HUBOT_SLACK_PROD_ROOM

      ccRoom = slackRoomMap[ccString] ? ccString

      return if ccRoom == room
      return if user == "Shell"

      robot.messageRoom ccRoom "#{ccText}"
```

For where I am in my Hubot development this is sufficient as I don't see any near future needs for more mappings.   Of course I would rather have a more elegant method for setting these rooms, maybe pass yaml/json in the ENV variable or some other good trick.  But I have to keep telling myself not to try to do to much.  If in the future I need a better method for mapping rooms I can deal with it then.

Next up is the ( hopefully ) last piece I need for a Hubot MVP ( Minimum Viable Product ) ... security.
