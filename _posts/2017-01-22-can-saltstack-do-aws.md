---
layout: post
title: "Can SaltStack do AWS, the VPC"
teaser: "We started on Rackspace back when it was still trying to compete with AWS in the IaaS. As a startup, Rackspace worked just fine.  But if a company grows and needs more and their IaaS vendor is not growing their offering, there comes a tipping point.  Security was our tipping point.  We could not get the security we needed without either expensive dedicated hardware or a way to convoluted network architecture.  And so we are moving to a new IaaS vendor, AWS.

To get started at AWS I created a VPC by hand:  subnets, routing tables, gateways, security groups. Add in a salt server and I am able to use Salt to spin up and manage my AWS servers."
tags: devops, saltstack, aws
header:
    image_fullwidth: "mountains.jpg"
---

 We started on Rackspace back when it was still trying to compete with AWS in the IaaS. As a startup, Rackspace worked just fine.  But if a company grows and needs more and their IaaS vendor is not growing their offering, there comes a tipping point.  Security was our tipping point.  We could not get the security we needed without either expensive dedicated hardware or a way to convoluted network architecture.  And so we are moving to a new IaaS vendor, AWS.

 To get started at AWS I created a VPC by hand:  subnets, routing tables, gateways, security groups. Add in a salt server and I am able to use Salt to spin up and manage my AWS servers. And I thought, like many others before me, "I can turn this infrastructure into a cloud formation template".   And I did, all 1793 lines of it, in JSON.  Since everything I do in SaltStack is YAML, I would much rather use YAML. And as I look I find that the AWS export tool provides JSON, but CloudFormation will take either YAML or JSON.  A web translator and 1/2 hour later I have a YAML file, and it is only 1214 lines long.  Removing 588 lines containing only start or end brackets/braces makes it more readable, but it is still 1214 lines I have to go through and change references into CloudFormation variables so that I can use this as a template for another VPC.  Which could take days ( per a friend that did this same thing it took 2 days ).

Should I invest two days into this with the results being a template that will create exactly one VPC design? Or should I try to use the native SaltStack states?  When I looked at this a couple of years ago the answer was to go with the CloudFormation template.  And when I look now I don't find a lot more out there to indicate people are using states.  I find a few questions on how to do a VPC using salt states, but It seems like more people are using the boto_cfn to run their CloudFormation template from within saltstack. And there are no saltstack formulas that actually create a VPC and the base components like routing tables, gateways, etc...

Are there no formulas because people can not make it work?  Or is it just that no one has taken the time to write one? Looking at SaltStack documentation it appears that the hard working SaltStack staff and volunteers have gotten all the pieces in place to make it possible to use states. So looks like option two. I guess I will have to try to write one.

And a couple of days later I have a formula that works.  Well, it works 95%.  I get an error when trying to use nat_gateway_subnet_name inside a boto_vpc.route_table_present.  So SaltStack issue created and I find a workaround until that error gets fixed.  Which means a 2 run solution.  First formula run creates all the pieces. Then copy the NAT Gateway ID into the pillar and re-run the formula to add the NAT Gateway as the default for internal subnet routing table rules.

Two days invested in creating an aws-formula is much better than creating an inflexible CloudFormation Template. And the [aws-formula](https://github.com/saltstack-formulas/aws-formula) is now live on salt-stack-formulas. The sample pillar shows the creation of a three tier, three Availability Zone VPC.  It creates:
* Key Pairs
* VPC
* Internet Gateway
* Subnets ( web, app, db X 3 AZ)
* NAT Gateways in the web subnet of each AZ
* Public and private routing tables for each AZ
* Add default routes with public and private routing tables ( Internet Gateway for public, NAT Gateway for private )
* Associate Subnets with Routing Tables
* Security Groups ( Web, App, Salt-master, openVPN, ipsec VPN)

There are no servers in this, but those will all be created by Salt. Sure wish we had VPC pairing between regions.  That would allow this to really be complete.  But alas not yet. So those ipsec based VPN servers still have to be created by Salt and then the routes added to the AWS pillar.
