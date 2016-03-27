---
layout: post
title: Chatops Hubot The Basics
---

Hubot is really not hard get started with.   There are many how-tos on this, but the basics are ( commands based on ubuntu 14.04 install)
1.  Install node.js ( I used 5.x version from deb.nodesource.com): `npm install node.js ???`
2. Install git-core, libssl-dev, redis-server: `apt-get install git-core libssl-dev redis-server`
3. Install coffeescript: `npm install -g coffee-script hubot yo generator-hubot`
4. create the hubot install: `yo hubot`
5. Start hubut: `cd hubot; bin/hubot`
6.  Stop Hubot:  `control-c`

And I had a hubot instance running.   I could give it commands on the command line and get responses.  help showed me all the things that come by default.  After trying everything out, I disabled several of the default items that I did not want ( like google translate.)  To remove one of the pre-bundled items you need to
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
2. Add the .env variable: `export HUBOT_SLACK_TOKEN=xoxb-XXXX-XXXXXX-XXXXXX-XXXXXX`
3. Start hubot with slack enabled:  `bin/hubot --adapter slack`

And all of a sudden hubot showed up in my slack as a bot.  Invite hubot to a room and then he will listen there.

Something that I found missing from hubot was the ability to use .env files like all my other node apps ( using dotenv or similar ).  Hubot can use a .env file but it requires `export` in front of each line.   I am using a modified node_modules/.bin/hubot to allow me to use "normally" formatted .env files.  That edit has been [submitted to github](https://github.com/github/hubot/issues/1153)
