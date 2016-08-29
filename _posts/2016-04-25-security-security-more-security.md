---
layout: page
title: Security, Security, More Security
teaser: "The security put in place last time was not enough for me.   While it provides that no one can affect production servers except those in the hard-coded Hubot admin group, I don't like that anyone who has a slack account can affect the test environment."
tags: hubot chatops slack security
header:
    image_fullwidth: "mountains.jpg"
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

```json
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

This module uses the slack user IDs for authentication.  To add a slack user to a group use their ID ( what you see after the @ in a mention in a room).   In the below mythical example these IDs are **firstlast**.  Here the developers and qa staff are added to roles.

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

Now to restrict commands/modules using these new user roles.  The method I decided to use was to create a restrictCommand global function which can be called from each module that needs to be restricted.

```javascript
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

It starts with allowing admins ( they are Gods after all so can use all modules ), followed by a simple loop to check for membership in a group.  If either of these if statements is true then the function return a **false** meaning the command should not be restricted.   Otherwise a true is returned (command is restricted), but not before adding some color to our hubot by having a set of famous quotes or snarky replies that hubot will randomly choose from.   Because it is part of the ChatOps ethos to not take ourselves too seriously.

Now the lines that go at the top of a module/command to call the above function.

```javascript
robot.respond /list (apps|applications) on (server)? *([\w-_]+)/i, (msg) ->
  allowedRoles = ["developer","qa"]
  return if robot.myCompany.restrictCommand(msg, allowedRoles )
  ...
```

The allowedRoles list is passed to restrictCommand.  If it returns a true then a **return** exits the robot.respond without running any of the rest of the robot.respond code.  The above example allow **/list apps** to be run by Admins, developers and qa while anyone else just gets our colorful message back.  Add these two lines ( varying what roles are in allwedRoles as needed ) to every module that needs restricting and you now have role based security in place.

Audit Note:  if this same pattern is used on all modules, a grep can be used to easily audit all of robot.respond command restrictions. From the hubot root directory run this command to return a list of robot.respond lines followed by the allowedRoles.

```bash
grep -A 1 -R robot.respond scripts/*
```
