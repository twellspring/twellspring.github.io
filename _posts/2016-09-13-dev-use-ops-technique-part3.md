---
layout: post
title: "Can Ops use Dev Techniques Part 3, Deployment Workflow"
teaser: "The last piece of the development process I want to look at is the Development Workflow. What is the process for getting the code from the developer and onto a physical server where customers can use it. Generally speaking companies have a set of environments that  code will flow through before it gets into production.  Lets keep this simple with three environments"
tags: devops, saltstack,
header:
    image_fullwidth: "mountains.jpg"
---
#### Piece 3: Deployment Workflow ####

The last piece of the development "process" I want to look at is the Development Workflow. What is the process for getting the code from the developer and onto a physical server where customers can use it. Generally speaking companies have a set of environments that  code will flow through before it gets into production.  Lets keep this simple with three environments:

* development ( dev )
* quality assurance ( qa )
* production ( prod )

After code is written it gets deployed to a dev server where the newly created features are tested individually to make sure they work as expected.  Then the code is promoted to a qa server where the overall application is tested to make sure new changes did not affect the rest of the application in ways we did not expect.  And once this is validated, the code goes to a prod server.

Can this also work for the operations automation code? We first need to work through differences environment definitions. My automation server deploys code to dev, qa and prod.  So this server is partly prod, partly qa, partly dev.  Anyone in Sysadmin or DevOps is used to this definition difficulty.  The environment lines are not so clear cut when it comes to Ops servers. But I think it is clear that our automation code needs to be tested.  So It is better to create a new set of environment definitions for the Ops servers/services.  Here are mine:

My SaltStack automation server has three environments, Lets call them test, stage, prod.  Test is where I try out the SaltStack pillars and states I am updating.  I create servers just for the purpose of testing these changes.  Since this is only for testing salt code, the developers do not have any access to this environment. I use this environment for both testing individual features and testing the overall application, but since it is done separate from the "real world" it it is roughly the equivalent of dev.

Once tested in dev, SaltStack code is moved to the stage environment.  All dev environment servers are in this automation environment, and so the automation code now in a real world with users and real code being run.  So from a environment standpoint this is roughly equivalent to qa.  It also means that either an automation change or a dev code change could be responsible for a Dev feature not working.  I have not found a way to work around this, and I don't think we really need to.  Because this is not really different than front end and back end engineers testing their new features together and having a problem.  The issue has to be researched to determine which teams code the problem is in.

Now that the SaltStack code is tested in "real world", it gets promoted to prod.  The SaltStack prod environment contains both the qa and prod servers.  Using the same automation code makes qa and prod are as similar as possible to ensure the validity of our qa testing before our Dev code goes to prod.

Using Ops specific environments, the automation code workflow looks very similar to the Dev code workflow.  And so once the answer to "can Ops use the same process as Dev" is theoretically yes.

To make git flow work practically for my automation code, I use branches corresponding to the described environments.  I have three directories on the salt server.  Each directory is a copy of the automation repository set to a different branch.  Then in my master config I use Salt **file_roots** to connect a directory to a salt environment and the **top** file to connect servers to the appropriate environment.  The result of using a branch per environment is that my git flow has three persistent branches ( develop, stage, master ) instead of the typical two ( develop, master ).

Just one more part of this series to go, where I look at the questions "Will Ops use the Dev processes?" and "Why is it important for Ops to do so?"
