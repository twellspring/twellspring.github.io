---
layout: post
title: Chatops and Hubot the basics
---
Hubot was really not hard get started with.   I started with it on my MacOS laptop.   I already had these dependencies installed

* node.js 5.x
* npm
* git

So what was left from the [hubot installation instructions ](https://hubot.github.com/docs/)

* Install hubot libraries/components: `npm install -g yo generator-hubot`
* Generate an instance of hubot: `mkdir hubot; cd hubot; yo hubot`
* Start hubot: `bin/Hubot`
* Stop hubot: `control-c`

And I had a hubot instance running.   I could give it commands on the command line and get responses.  help showed me all the things that come by default.  After trying all the default commands out, I disabled several items that I did not want ( like google translate.)  To remove one of the pre-bundled items you need to

1. Remove the item from package.json
2. Delete the node_modules/xxxxx directory of the same name
3. Remove the entry from `external-scripts.json`
4. start/restart hubot

There are many additional plugins available at [hubot scripts](https://github.com/hubot-scripts).  Two that I installed are

* hubot-auth - user roles to be used for command authorization
* hubot-diagnostics - some tools to help with basic troubleshooting

To install a plugin

1. `npm install  --save xxxxxx`
2. edit `external-scripts.json` and add the plugin
3. start/restart hubot


Now to get slack working.  I had my slack admin create a hubot integration and give me the token.   Then it is just

1. Install the plugin: `npm install --save hubot-slack`
2. To my environment add the slack token: `export HUBOT_SLACK_TOKEN=xoxb-XXXX-XXXXXX-XXXXXX-XXXXXX`
3. Start hubot with slack enabled:  `bin/hubot --adapter slack`

And all of a sudden hubot showed up in my slack as a bot.  Invite hubot to a room and then he will listen there.

Something that I found missing from hubot was the ability to use .env files like all my other node apps ( using dotenv or similar ).  Hubot can use a .env file but it requires **export in front of each line.   I am using a modified node_modules/.bin/hubot to allow me to use "normally" formatted .env files.  That edit has been [submitted to github](https://github.com/github/hubot/issues/1153)

With it, my .env now has three variables

```
HUBOT_SLACK_TOKEN=xoxb-XXXXXXXXXXX-XXXXXXXXXXXXXXXXXXXXXXXX
HUBOT_AUTH_ADMIN=1
HUBOT_SALT_AUTH=XXXXXXXXXXXXXXXXXX
```

The First variable I have already talked about.  The remaining two will be covered in the next two posts about Saltstack and Security.
