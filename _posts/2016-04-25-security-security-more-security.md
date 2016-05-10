---
layout: post
title: Security, Security, More Security
---

The security put in place last time was not enough for me.   While it provides that no one can affect production servers except those in the hard-coded Hubot admin group, I don't like that anyone who has a slack account can affect the test environment that our developers rely on.    So we need user groups with different levels of access.   I think 3 will do for demonstrating group access:

1. God ( Admins )
2. Developers
3. QA
4. Everyone else

The [hubot-auth module](https://github.com/hubot-scripts/hubot-auth) will provide us the ability to make these groups and as many more as we may need.  First lets install it.
Add hubot-auth to your package.json file:

```
npm install --save hubot-auth

```

and then add hubot-auth to external-scripts.json:

```
[
  "hubot-help",
  "hubot-pugme",
  "hubot-redis-brain",
  "hubot-rules",
  "hubot-auth",
  "hubot-shipit"
]
```

There are now several new commands:

```
! <user> doesn't have <role> role - Removes a role from a user
! <user> has <role> role - Assigns a role to a user
! what roles do I have - Find out what roles you have
! what roles does <user> have - Find out what roles a user has
! who has <role> role - Find out who has the given role
```

This module uses the slack user IDs for authentication.  To add a slack user to a group use their ID ( what you see after the @ in a mention in a room).   In the below mythical example these IDs are <firstname><lastname>.  Here are the developers and qa added to roles.

```
! malreynold has developer role
Hubot: @rivertam: OK, malreynold has the 'developer' role.
! zoewashburne has developer role
Hubot: @rivertam: OK, zoewashburne has the 'developer' role.
! washwashburne has developer role
Hubot: @rivertam: OK, washwashburne has the 'developer' role.
...
! shepherdbook has qa role
Hubot: @rivertam: OK, shepherdbook has the 'qa' role.
```

Now to restrict these new user roles.  The method I decided to use was to create a restrictCommand global function.  Then in each module we create the first lines will specify what roles can use that module.

```
restrictCommand: (msg,allowedRoles) ->
  return false if robot.auth.isAdmin(msg.envelope.user)
  for allowedRole in allowedRoles
    return false if robot.auth.hasRole(msg.envelope.user,allowedRole)

  replies = [
    "Sorry, Charlie.",
    "But being this is a .44 Magnum, the most powerful handgun in the world
    and would blow your head clean off, you've gotta ask yourself one question:
    'Do I feel lucky?' Well, do ya, punk?",
    "I'm sorry, Dave. I'm afraid I can't do that.",
    "Sometimes 'No' is the kindest word. I'm being kind now.",
    "Set phasers to stun."
    ]
  msg.send msg.random replies
  return true
```

It starts will allowing admins ( they are Gods after all ), followed by a simple loop to check for membership in a group.  If either of these match, we return a **false**, which means the command is not restricted.   If one of these does not pass we return true, but not before adding some color to our hubot by having a set of famous quotes or snarky replies that hubot will randomly choose from.   Because it is part of the ChatOps ethos to not take ourselves too seriously.

All that is left is to add a couple of lines to the top of a module/command we want to restrict.

```
robot.respond /list (apps|applications) on (server)? *([\w-_]+)/i, (msg) ->
  allowedRoles = ["developer","qa"]
  return if robot.myCompany.restrictCommand(msg, allowedRoles )
  ...
```

If restrictCommand returnes a true, then the command is exited.  So this command will restrict the **/list apps** module to Admins, developers and qa and anyone else gets a message back and the command does not run.  Add these two lines ( varying what roles are in allwedRoles as needed ) to every module that needs restricting.   If the same pattern is kept on all modules, a grep can be used to easily audit all of robot.respond command restrictions:

```
grep -A 1 -R robot.respond *
```
