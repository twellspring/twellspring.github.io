---
layout: post
title: Trust but verify? Maybe, but better to not trust and sanitize
teaser: "It would be nice if the old Regan slogan, which is an even older Russian proverb, was sufficient for our chatops security mantra.  I would like to be able to always trust the system's users, especially because chatops users are trusted employees."
tags: hubot chatops security
---

It would be nice if the old Regan slogan, which is an even older Russian proverb, was sufficient for our chatops security mantra.  I would like to be able to always trust the system's users, especially because chatops users are "trusted" employees.  And in most cases they are the developers I work with and along side daily.   But alas, I don't think I can leave it at that.  There are too many instances in the news of fraud and data breaches being perpetrated by an insider i.e. a trusted employee.   I want to empower the developers I work with, but I don't want to see my chatops implementation becomes an open attack vector used to perpetrate data theft.    Instead I am going to not trust and sanitize.

Sanitizing data is common practice.  Any data that comes in from a user needs to be sanitized.  Not most of the data ... ALL data.   It only takes one un-sanitized field on a website to enable code injection.  Most Developers are aware of this and are diligent with ensuring all data is sanitized, but DevOps and Operations staff might not be in that same mentality when it comes to their internal tools like chatops.   So lets be good security minded developers and sanitize the data coming into our chatops implementation.

Data comes in to our chatops system via our **robot.respond** and **robot.hear** statements. Here is possibly the most used command in chatops

```
robot.respond /pug me/i, (msg) ->
```

Nothing hard here ... if the phrase "**hubot pug me**" is sent to chatops it returns a picture.  This does not have a potential for code injection, because if someone tried an injection "**`hubot pug <script>alert('Injected!');</script> me`**" it would not match our robot.respond criteria and so hubot would not respond to it.

To continue the pug me example, lets look at the **pug bomb** command ( slightly altered to help illustrate the point).

```
robot.respond /pug bomb( (.*))?/i, (msg) ->
  count = msg.match[2] || 5
  msg.http("http://pugme.herokuapp.com/bomb?count=" + count)
```

This command allows the user to pass a number and that number is added to a URL.  But the pattern **.*** will match any string meaning a user can put something besides a number and in the next two lines that something is added to the herokuapp.com URL.   A large vulnerability?  Maybe not in this example, but allowing unsanitized data gives the user the ability to affect things they should not be able to.   And if any of those users just happen to be javascript experts, they can probably find a way to use something like this against us.

The robot.respond ( and robot.listen ) lines are javascript regular expressions.  Take a look at w3schools.com  [Javascript Regular Expressions](http://www.w3schools.com/jsref/jsref_obj_regexp.asp) to see the options and to test out regular expressions.  In place of the **.*** we can use **\d** to match only digits.  This gives us the code as it actually exists in the hubot-pugme module.

```
robot.respond /pug bomb( (\d+))?/i, (msg) ->
  count = msg.match[2] || 5
  msg.http("http://pugme.herokuapp.com/bomb?count=" + count)
```

This data is sanitized because the (\d) pattern match will only allow digits.  A user can send a really big number which could possibly be used to invoke a Denial of Service attack, but they can not do script injection because that line would not match and hubot will ignore it.

A little more practical example is matching a server name.  Server names per the DNS spec can only contain letters, numbers and the - symbol.  Here is a command to do that.

```
robot.respond /list (apps|applications) on server ([\w-]+)/i, (msg) ->
```

**[\w-]+** will match all those characters.  It also matches the _ character.  So this regex could also be written more precisely as **[a-z0-9-]+**.  In either case our input is once again sanitized.   So go check all of your robot.respond & robot.hear lines to make sure ALL input data is sanitized.


**Note1** the trailing dash in both of these last two regex expressions is not interpreted as a range character as it is the final character in the regex and a range character has to have something on either side of it.  It could also be escaped **\\-**

**Note2**  If your hostnames are well formed, you can make this regex even more specific.  This additional level specificity is not improving the data sanitizing but it will mean that the user can not use names that do not conform to naming patterns.  So for example if the naming scheme is:  datacenter-business_unit-type-environment-number ( example: west-mktg-web-prod-3 ), the regex could be **[\w]+\\-[\w]+\\-[\w]+\\-[\w]+\\-[\d]+**
