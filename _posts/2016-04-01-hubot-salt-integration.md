---
layout: post
title: Hubot Saltstack Integration
---

Before finding ChatOps our strategy was to create a salt-api in node.js that exposed just the needed parts of Saltstack via a RESTful API that returns JSON.  This would allow a developer to integrate Saltstack with our internal management tools with minimal understanding of salt. Much of the time the salt commands returned JSON that could be passed directly through the salt-api which made many salt-api commands easy to implement.  But there are cases where I found it beneficial to run multiple salt commands and aggregate data or to limit what part of the JSON data returned by salt gets returned through the salt-api.

When ChatOps "arrived", the decisions for the above methodology still made sense.  In the future the Saltstack and Hubot development might not always be done by the same person so, like above, this allows the Hubot developer to not have to know Saltstack intimately. And our internal salt-api provides a control layer to strictly limit what ChatOps can do.  This is important as at this point I am not confident enough in Hubot's security design & implementation to trust it with full access to all salt commands and by extension to all facets of production servers. ( But as it is open source, I will be looking into this in the future. )

And so to integrate Hubot with my salt-api was a standard CoffeeScript http call.  The pattern for my Hubotized salt commands follow this pattern

* robot.respond
* http request to salt
* error checking
* response to user

Here is some boilerplate code for getting information about a server from salt-api
( if needed, scroll left/right in text box to see full lines)

```
robot.respond /Get information about server *([\w-_]+)/i, (msg) ->
  serverName = msg.match[1]
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
        msg.reply "There was an general error:\n#{err} getting data aboug #{serverName}"
```

Though this code is specific to the internal salt-api, the same methodology works for accessing Saltstack API.  And doing POST/PUT requests just requires changing the method and passing the appropriate body.

```
robot.respond /deploy ([\w-]+) to ([\w-]+) version ([\w-_/\.]+)?([\w-_]+)?/i, (msg) ->
  appName     = msg.match[1]
  serverName  = msg.match[2]
  appVersion = msg.match[3]

  postData = JSON.stringify({
    version: appVersion
  })

  httpURL = "#{saltUrl}/server/#{serverName}"/app/#{appName}"

  msg.http(httpURL)
    .headers('Content-Type': 'application/json', 'Authorization': saltAuth)
    .put(postData) (err, res, body) ->
      try
        jsonBody = JSON.parse(body)
        if jsonBody.result != "failure"
          msg.reply "Successfully deployed #{appName} to #{serverName}"
        else
          msg.reply "Error deploying #{appName} to #{serverName}"
      catch error
        msg.reply "There was an general error:\n#{err} deploying #{appName} to #{serverName}"
```

My nascent Hubot skills are sufficient now to create a MVP ( minimum viable product ) for my company ChatOps.   
For my initial ChatOps deploy the MVP is

* list types of servers
* list all servers of a given type
* Get information about a particular server ( ip, version of app installed, .... )
* Deploy an app branch/tag/hash to a server

For the longer requests like deploy I decided to add in a msg.send line before the msg.http.  This gives immediate feedback to the user so they know that Hubot is on the job.  Something like:

```
msg.send "Deploying #{version} to #{server}. \nIf this takes a few minutes, it is probably due to a slow `npm install`."
```

And with the above in place, I introduced ChatOps to my developers in a new slack room dedicated to hubot.   Many commented on how cool this was, and they played with the commands.  But it took a few days until people started really using the above server commands.   It quickly became apparent that having all Hubot commands in that one room made a big mess.  The goal of this room was to have a log of changes being made via Hubot, but with all the `pug me`, `ship it` and `help` commands it was quite unreadable.  So I went looking for how our dedicated room could be brought back to the goal.
