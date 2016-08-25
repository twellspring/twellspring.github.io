---
layout: post
title: Chatops at 6 months. How has it changed development. After
teaser: "In my last post I looked at where the development process was before ChatOps was discovered and implemented. My how things have changed in these six months.  Developers can do a lot more now."
tags: hubot chatops slack saltstack
---

In my [last post](/blog/chatops-at-6months-before/) I looked at where the development process was before ChatOps was discovered and implemented. My how things have changed in these six months.  Developers can do a lot more now.

#### A developer can ( with a single Slack command ) ####
* Get a list of development servers
* Get a list of development environments
* Get details for all servers in a development environment
* Create a single server fully configured, on the correct code branch.
* Create a full environment with all needed servers fully configured and on the correct code branch. Web servers would be automatically pointed at the api server in the set.
* Deploy new code branches to a server
* Terminate a single server
* Terminate an environment

#### A developer can sometimes not ####
* Get the syntax for the Slack command correct
* Troubleshoot why a particular server build failed

#### A developer can not ####
* Update the environment file ( .env ) for an application
* Deploy a new application

#### Server Testing Process ####
Like before, after working on code locally the developer moves that code to a server environment.  But the process of doing that is much simpler now.

* Figure out what servers and code branches they needed for testing.
* Run a Slack command to create the needed environment.
* Proceed with testing.
* Run a slack command to terminate the environment

#### Thinking Process ####

In the last few weeks I have been seeing the thinking process of many developers getting close to this:

* Creating a server development environment is easy
* I can wait until I am ready for testing to create a server environment
* If I need to make any changes, just terminate the environment and create a new one.
* When I am done testing I can terminate the environment

I say **getting close to** because thinking and habits don't always change quickly.  Some of the developers are fully at the above thinking process.  Others are somewhere between the before ChatOps thinking and the above thinking.  Transitioning people's thinking process, even if the change is to their benefit, still takes effort and time.  The transition of my developers thinking process was no different.  Next post will describe the transition process and what is needed to make sure it is successful.

#### "Can Not" Notes ####
**Troubleshoot why a particular server build failed** is was initially hard for a developer as ChatOps would return all the SaltStack failed states. This meant a simple error like incorrect branch would return 50+ lines of SaltStack JSON output due to all the states that failed because they were dependent on the git state. Filtering out these dependent failures cut out the noise.  Now a developer can pretty easily interpret the most common failure, typo in the branch to deploy, without understanding SaltStack or even knowing that the error message comes from SaltStack.  Here is an example:

```
{
  "git_|-xxxxxxx-repo_xxxxxxxxx_|-git@github.com:xxxxxxxx/xxxxxxxx.git_|-latest": {
    "comment": "No revision matching 'hotfix/x.xx.xx' exists in the remote repository",
    "name": "git@github.com:xxxxxxx/xxxxxxx.git",
    "start_time": "20:34:56.757768",
    "result": false,
    "duration": 489.839,
    "__run_num__": 3,
    "changes": {}
  },
```

Failures not caused by command syntax problems are usually not as clear and require Ops help to troubleshoot.  Fortunately these do not happen very often.

**Deploying new applications** and **updating environment files** are purposely not within the developer's abilities.  The server development environment is a intermediate step between development and production. Therefore it is controlled to ensure it is a similar as possible to production and the tests run in this environment are valid tests of what will happen when this code gets deployed to production.
