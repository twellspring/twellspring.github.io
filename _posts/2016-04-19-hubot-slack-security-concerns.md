---
layout: page
title: Hubot and Slack Security concerns
teaser: "The last post used middleware to restrict what room commands can run it.  But there is still no restrictions on who can use a command and what server they can run it on."
tags: hubot chatops slack security
header:
    image_fullwidth: "mountains.jpg"
---

The last post used middleware to restrict what room commands can run it.  But there is still no restrictions on who can use a command and what server they can run it on.   It does not take my 10 years of work at financial companies and countless audits to know that this level of open access is not a good idea.   So before Hubot gets implemented I want to have a minimal level of security.   Remember I am going for MVP here, not what I think will be the end all security design a year from now.   Walk before running.

The minimum I see is:

1. Admins can run commands like deploy on any server
2. Non-admins can run commands like deploy only on testing servers

Fortunately all my servers have well defined names and the environment of a server can be easily parsed out of the name

```
server_split = server_name.split "-"
server_env = server_split[3]
```

Now I just need put a check at the top of the deploy command that will check the above server_env.   I created a global function to validate the above two conditions are met:

```
module.exports = (robot) ->

  class myCompany

    restrictServer: (msg,server) ->
      user = msg.envelope.user
      server_split = server_name.split "-"
      server_env = server_split[3]
      return false if server_env == 'test'
      return false if robot.auth.isAdmin(user)

      msg.send "You are not authorized to run this command on #{server}"
      return true
```

The two lines starting with **return false** are my conditions.   If either of these conditions are met the function returns false, which means the command should not be restricted.   If both are not true it sends a message and returns true.  The first checks if the server is a test server.  If so, anyone can deploy to it.   The second checks if the user is an Admin, which is defined in the environment when starting up Hubot.

Now at the top of my deploy command ( and any other command that needs this security)  I put in a check

```
robot.respond /deploy ([\w-]+) to ([\w-]+) version ([\w-_/\.]+)?([\w-_]+)?/i, (msg) ->
  appName     = msg.match[1]
  serverName  = msg.match[2]
  appVersion = msg.match[3]

  return if robot.myCompany.restrictServer(msg, serverName )
  ...
```

To test this I used local Hubot.  Adding `HUBOT_AUTH_ADMIN=1` to the environment means the Shell user is an admin and `HUBOT_AUTH_ADMIN=2` ( or any other value ) means Shell is not.  This allowed me to validate the above restrictions worked as I expected. Then when I deploy this to my production server and run it with Slack I will add my Slack ID to the `HUBOT_AUTH_ADMIN` variable.

Just to clarify, restrictServer being true will cause the `return` line above to exit the robot.respond function and not process the rest of the command.   This is what we want.  Only if restrictServer returns false ( do not restrict server ) do we want to continue.   And the only way restrictServer returns false if one of the two conditions I set forth at the beginning of this article is met.

After testing this thoroughly locally I was ready to deploy Hubot to Slack ( without telling anyone ) so that I could test it there.  Then when I was confident that users could only affect test servers I told the developers about Hubot and let the flood of **pug me** command commence.

MVP done and ChatOps 1.0.0 released to my company with much excitement, and a week or two later developers start to really use it.   Life is pretty good.   But of course being a devops/sysadmin/linux/unix guy I am not satisfied with this and want to improve upon it.  And that 10 years of Financial screams at me for the first improvement to be in security.   Sigh, I guess I can't ignore that voice.   So better security is up next, and I think we need some kind of role based authentication/authorization.   Slack takes care of the authentication and we can use the hubot-auth model for the roles and authorization.
