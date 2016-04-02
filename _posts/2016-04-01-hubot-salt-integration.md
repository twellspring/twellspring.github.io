---
layout: post
title: Hubot Saltstack Integration
---

Before finding ChatOps the strategy was to create our own salt-api in node.js that exposed just the needed parts of Saltstack via a RESTful API.  This would allow a developer to integrate Saltstack with our internal management tools with minimal understanding of salt. It also allowed for aggregation/wrapper functions to present the data in a better way than was available from the Saltstack API. When ChatOps "arrived", the decisions for the above methodology still made sense.  In the future the Saltstack and Hubot development might not always be done by the same person. And our internal salt-api provides a control layer to strictly limit what ChatOps can do as I am still not confident enough in Hubot's security design/implementation to trust it with full access to all salt commands and by extension to all facets of production servers.

The actual implementation of a salt command follows this pattern

* robot.respond
* http request to salt
* error checking
* response to user

Here is some boilerplate code for getting information about a server from salt-api
( if needed, scroll left/right in text box to see full lines)

```
robot.respond /Get information about server (server)? *([\w-_]+)/i, (msg) ->
  deployApp = msg.match[1]
  httpURL = "#{saltUrl}/server/#{serverName}"
  msg.http(httpURL)
    .headers('Content-Type': 'application/json', 'Authorization': saltAuth)
    .get() (err, res, body) ->
      try
        jsonBody = JSON.parse(body)
        if jsonBody.result != "failure"
          textBody = JSON.stringify(jsonBody, null, 2)
          msg.reply "Here is information about `#{serverName}`\n```#{textBody}```"
        else
          msg.reply "I could not find any information.  Are you sure `#{serverName}` is a real server?"
      catch error
        msg.reply "There was an general error:\n#{err}"
```

Though this code is specific to the internal salt-api, the same methodology works for accessing Saltstack API.

For my initial ChatOps deploy the MVP ( Minimum Viable Product) was

* list types of servers
* list all servers of a given type
* Get information about a particular server ( ip, version of app installed, .... )
* Deploy a new app branch/tag/hash to a server

For the longer requests like deploy add in a msg.send line before the msg.http.  This gives immediate feedback to the user so they know that Hubot is on the job.  Something like:

```
msg.send "Deploying #{version} to #{server}. \nIf this takes a few minutes, it is probably due to a slow `npm install`."
```

And with the above in place, I introduced ChatOps to my developers in a new slack room `staging`.   Many commented on how cool this was, and they played with the commands.  But it took a few days until people started really using the above server commands.   It quickly became apparent that having all Hubot commands in that one room made a big mess.  The goal of this room was to have a log of changes being made to staging, but with all the `pug me`, `ship it` and `help` commands it was quite unreadable.  So I went looking for how `staging` could be brought back to the goal.
