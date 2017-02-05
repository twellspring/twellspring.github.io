---
layout: post
title: "AWS MySQL Choices:  MySQL, RDS, Aurora oh my"
teaser: "My migration to AWS is to the point where I have to decide on what database to use.  Or should I say what version of a database to use.  Our current DB is MySQL, so I want to stick with compatible options.   And there are three:  self-managed MySQL, RDS MySQL, RDS Aurora.   Which to choose?"
tags: devops, saltstack, aws
header:
    image_fullwidth: "mountains.jpg"
---

My migration to AWS is to the point where I have to decide on what database to use.  Or should I say what version of a database to use.  Our current DB is MySQL, so I want to stick with compatible options.   And there are three:  self-managed MySQL, RDS MySQL, RDS Aurora.   Which to choose?

My Ubuntu servers running self-managed MySQL perform their job just fine.  Our only significant outage in the last year that was DB related was running out of space (oops, my bad).  So from a basic reliability standpoint there is no problem with self-managed MySQL.  The issues come in when you add in backups & replication, which is a given for most organizations.  Developing the processes and procedures for performing backups and for creating slaves took work.   And it takes ongoing work.  Replication issues have caused me to rebuild two MySQL Slaves this year alone and many more last year.  So I have that process down. But I would rather not have to deal with it at all. The story is similar with Backups.   Time fixing backup issues and replication failures account for over 50% of production maintenance.   I want to outsource all of this backup & replication maintenance.   And both RDS MySQL and RDS Aurora do that.  Backups ... check the box.  Creating a slave in another AZ ... check the box.  Oh and lets add in Encryption ... check the box.

So RDS MySQL or Aurora?

RDS is MySQL with all of the automation/maintenance I am looking for in place.  The 20-30% cost increase of an EC2 instance of the same size is well worth the cost for not having to do all that maintenance.   But in the end it is just MySQL servers that have a lot of automation build around them.  Aurora on the other hand is built differently: 6X data replication, dedicated storage subsystem, rapid replica creation.  Maybe call this Oracle RAC Lite.  Which to me is a good thing.  I managed Oracle RAC for many years and the maintenance efforts required was way higher than a similar number of MySQL servers and slaves. RAC was cumbersome and required specialized DBAs to keep it running and of course cost many arms and legs.

Technically Aurora looks to be a better solution than RDS MySQL. So the decision gets down to price.   Can I afford Aurora.   When I first looked at Aurora, the answer was no, but that was primarily because there were no smaller instance sizes.   r3.large for about $210/month was small enough for my production instances, but too much ( and too costly ) for development/test instances and special purpose slaves.  And I would rather not manage PROD in Aurora and Dev/Test in RDS MySQL.  Looking more recently I see that t2.medium was now an option with a $60/month price point.  So I need to re-evaluate.

And that re-evaluation lead me to go over the Aurora documentation much more thoroughly. And when I did I found way down at the bottom of the pricing page

```
I/O Rate	$0.200 per 1 million requests
```

Wow good catch.  No longer is a DB server a fixed cost ( add up instance and storage cost ).  Looking online I see many people have been surprised by a large and unexpected I/O cost on their first bill.  Not going to make that mistake.  An AWS support chat asking for help with estimating Aurora IO based on my current MySQL servers did not give me much information on how they might be different. So I am going to assume MySQL IOPs and Aurora IOPs will be about the same.  A quick calculation show me that an average of 100 IOPs/sec over a month adds about $50 in I/O costs.   Looking at my monitoring ( NewRelic ) shows me only % utilization, so guess I have to go old school:  install and enable sysstat.   I'll let you know how it turns out.
