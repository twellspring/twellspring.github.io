---
layout: post
title: "Can SaltStack do AWS, what's in a name"
teaser: "Two days of aws-formuala being live and I found it was broken.  Because of names. More precisely the collision of of names in different VPCs.  Aren't VPCs completely independent?  Well yes, except for names it appears."
tags: devops, saltstack, aws
header:
    image_fullwidth: "mountains.jpg"
---

Two days of [aws-formula](https://github.com/saltstack-formulas/aws-formula) being live and I found it was broken. Because of names. More precisely the collision of of names in different VPCs.  Aren't VPCs completely independent?  Well yes, except for names it appears.

The normal methodology for a salt state is to check if something needs to be updated and then update it if it does.  It appears that this  methodology is breaking down in the AWS state/module.  I think this is partly due to how AWS is structured and part to difficulties with the boto module.

The aws-formula uses object names to create all objects.  One reason for this is the lack of a mechanism for capturing the results of one state ( create Internet Gateway ) and using a part of that captured value ( Internet Gateway ID) as an input for another state ( create Route Table ).   Ansible's `Registered Variable` can accomplish this, but it is somewhat messy as the registered variable has to be manipulated in a command in order to parse out the part of the output desired, then that parsed out value captured in another variable before being used in subsequent commands.  Using Names instead of IDs for all AWS objects is a much cleaner methodology for creating a VPC and all its required components.  It removes all the complexity of parsing IDs based on state results.

AWS uses a tag to capture the name for objects.  In VPCs, this includes: Subnets, Route Tables, Internet Gateways, Network ACLs, Security Groups.   Multiple objects of these types can have the same name. Here is an example:

[AWS routing tables in same VPC with the same name](/assets/img/aws-route-table-duplicate-name.png)

The boto modules appear to get caught by this as well.  I have two VPCs in the same region.  Neither have the routing table named my_route_table.

[two vpc](/assets/img/aws-two-vpc.png)

[no my_route_table ](/assets/img/aws-no-route-table.png)

But yet when I run the following states instead of getting both to succeed, only one succeeds.

```yaml
boto_vpcProdEast5_my_route_table:
  boto_vpc.route_table_present:
    - name: my_route_table
    - vpc_name: vpcProdEast5
    - key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    - keyid: XXXXXXXXXXXXXXX
    - region: us-east-2

boto_vpcProdEast6_my_route_table:
  boto_vpc.route_table_present:
    - name: my_route_table
    - vpc_name: vpcProdEast6
    - key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    - keyid: XXXXXXXXXXXXXXX
    - region: us-east-2
```
The results show that the second state does not run because there is already a routing table by that name present, the one in the other VPC.

```yaml
----------
          ID: boto_vpcProdEast5_my_route_table
    Function: boto_vpc.route_table_present
        Name: my_route_table
      Result: True
     Comment: Route table my_route_table created.
     Started: 04:52:35.891737
    Duration: 1887.427 ms
     Changes:
              ----------
              new:
                  ----------
                  route_table:
                      rtb-1224827b
              old:
                  ----------
                  route_table:
                      None
----------
          ID: boto_vpcProdEast6_my_route_table
    Function: boto_vpc.route_table_present
        Name: my_route_table
      Result: True
     Comment: Route table my_route_table (rtb-1224827b) present.
     Started: 04:52:37.779406
    Duration: 239.461 ms
     Changes:

Summary for local-core-api-prod-1
------------
Succeeded: 2 (changed=1)
Failed:    0
------------
Total states run:     2
Total run time:   2.127 s
```

Since the state requires both the routing table name and the VPC it is in, I would expect the above to work since the combination of vpc_name and name ( routing table name ) is unique. But looks like it is not. The method route_table_present in states/boto_vpc.py is being called, which eventually calls _find_resources in modules/boto_vpc.py.   _find_resources is only using the name tag to filter the list of routing tables.
So until the filter also includes the VPC name, we will not be able to have the same object name in multiple VPCs.

In the past I have sometimes used globally unique names ... Security groups is an example.  These needed to be globally unique since a Security Group can be referenced across a VPC Pairing. Guess I need to make this more general   So for now making all VPC object names globally unique is going to be the SaltStack best practice for boto_vpc states.

[aws-formula](https://github.com/saltstack-formulas/aws-formula) has been updated to include this best practice.  All states will automatically append the VPC Name to all object names so you don't have to clutter the pillar with repeat reverences to the VPC Name.
