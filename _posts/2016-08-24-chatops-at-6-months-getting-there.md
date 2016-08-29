---
layout: page
title: Chatops at 6 months. How has it changed development. Getting There
teaser: "Using ChatOps to move from Ops-Serve to Self-Serve obviously took some technical work.  But as stated before I am focusing on the developer in this series, so I want to look at what it took to transition the development team to using this new tool."
tags: hubot chatops slack devops
header:
    image_fullwidth: "mountains.jpg"
---

Using ChatOps to move from Ops-Serve to Self-Serve obviously took some technical work.  But as stated before I am focusing on the developer in this series, so I want to look at what it took to transition the development team to using this new tool.

Changing our thinking and consequently our actions always takes work.  So as the implementer/expert/guru of something new I need to have a plan.  What can I do to help my team members in their process of thinking change? For me it means getting to the aha moment, where something clicks and I get the first glimpse of the bigger picture.  That glimpse helps drive me forward in learning and exploring, leading to thinking change.  It might not be the same aha for everyone, but I think we usually have a tipping point where our driving force changes and becomes more internal.

An important thing to realize and plan for is the process of change takes time.  I implemented ChatOps and told the team about the new tool and what it could do.  Some people were excited and started trying it out right away. But not everyone did. Both are to be expected and both are fine. It is much better to give people the freedom to move at their own pace and to get to their personal aha moment than to force something on them, particularly forcing it on them "for their own good."  Since this tool was designed to help my team mates, I want them to see the value in it and use it because it is valuable.

Now the tool is launched, the team knows about it and some are trying it out.  Time to put extra effort into those early adopters. For me this generally meant three things:

1. Talking with them and learning what they want to do, then giving them example commands to do it.
2. Looking at the log of ChatOps commands to see what they were doing and giving them suggestions at how to do it better.
3. Looking at the log of invalid commands to see what they were not able to do because the syntax was unclear.  I would help them with the syntax and then try to update the ChatOps help so the syntax was clearer.

Just doing these three things over a month or two helped the early adopters see the value in the tool and how it could improve their workflow.  This is thinking change.  It was usually followed by change in action, which in this case was incorporating ChatOps into their workflow.  Doing the above also lead to thinking change in me.  When I learned how people wanted to do something my brain would churn on how to change existing commands or add new ChatOps commands to make it easer, and then I would go make the changes.  And those improvements helped the early adopters to aha faster and to ask if it could do xxxx.  And this circle continued.

Once there was a few people using ChatOps as part of their development workflow, my role became much less active.  These early adopters used ChatOps as they collaborated, meaning the other team members got practical demonstrations of ChatOps.  It did not take many interactions like that before most of the remaining people started trying out ChatOps. And for the most part all I had to do was help people with syntax and making occasional suggestions on better ways to do things.

Of course there were some people who did not use it for many months.  Some were "too busy" and did not really try to use it until they had breathing space.  Others might have a role where creating development server environments was not part of their normal job, or someone else always did it so they did not have to. Whatever the case this is to be expected.  At this point most everyone has transitioned their thinking and workflow to the new tool, so a few late adopters not really using it is not a concern.

One significant item not mentioned above that was key to ChatOps success:  Fast Bug Fixes. Developers had sympathy for me because they are familiar with new implementations and finding bugs in new code.  But they are also under pressure to perform in their jobs.  So if I give them a tool and they find it does not work as needed due to bugs that are not fixed quickly, they might decide it is not worth the time and effort to learn a new tool AND to work around the bugs in that tool. Which would set the adoption process back with those important early developers. And so any time they discovered a bug I would give it high priority and usually have it fixed the same day.
