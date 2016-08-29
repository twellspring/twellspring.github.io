---
layout: page
title: Saltstack and a well formed hostname keeps environments clean
teaser: "The well formed hostname in my last post helped connect servers to the appropriate pillars/roles while keeping the top file simple.  One part of the well formed hostname used, three to go."
tags: saltstack
header:
    image_fullwidth: "mountains.jpg"
---

The well formed hostname in my last post helped connect servers to the appropriate pillars/roles while keeping the top file simple.  One part of the well formed hostname used, three to go.  Another foundational piece of a salt ecosystem is environments, and we can use another name part to facilitate environments.

I have three salt environments:  test, stage, prod.  My basic salt development cycle is to develop new features in test which only has test minions used for testing new salt functionality.  Once I consider a feature "complete" it gets committed to my salt repo's stage branch, which is used to apply the changes to minions in our staging environment via a highstate.  If there are no problems the feature gets committed to my salt repo's master branch and applied to minions in our prod environment via highstate.

<sup>**Note**: SaltStack's default environment is **base**, git's default branch is **master** and  my   datacenter's default environment is **prod**.  So I have to know the correct "default" to use in the correct place, but they all are the same thing: production.   The git and salt environment defaults could be changed to prod so all three would match, but since I don't really see them often besides when I first setup a server, it has not been high priority.

To setup environments so the above development cycle can work we need to update the **file roots** and the **top files**.


###  File Roots ###
The file roots are used to define the different environments.  Here is the file roots configuration in its own modular file in /etc/salt/master.d

**/etc/salt/master.d/file_roots.conf**

```
fileserver_backend:
  - roots

file_roots:
  test:
    - /data/repos/test/state
  stage:
    - /data/repos/stage/state
  base:
    - /data/repos/master/state

pillar_roots:
  test:
    - /data/repos/test/pillar
  stage:
    - /data/repos/stage/pillar
  base:
    - /data/repos/master/pillar
```

The file_roots section tells salt where to find the state files for a given environment.  Similarly pillar_roots tells salt where to find the pillar files for a given environment.  Each of the /data/repos folders is the same git repository, but with a different branch checked out.  This git repository contains /pillar and /state top level directories and all the pillar/states are within. master is set to the master branch, stage to the stage branch.  Test is pointed to whatever feature branch I am currently working on.

I looked at the idea of having the stage and master branches independent.   Doing this would allow code common to both stage and prod gets committed to both branches and code specific to one branch is committed to only the relevant branch.  But this went against [git flow](http://nvie.com/posts/a-successful-git-branching-model/) which is used by everyone else in my dev team.   So instead I decided to stick with git flow here with the addition of a persistent stage branch. Hence the workflow I described above.  Another method can be used to separate common code and environment specific code and will be a future post.

###  Top Files ###

There are now 3 different top files instead of the one from last post, but the contents of each is still simple.  Before I had figured out what I am describing, there was just one top file and was much more complicated. Frequently when implementing new functionality I had to make changes to the top file.  Migrating those changes from test to stage to prod would mean sometimes a particular section of the top file was different in the staging and prod branches. And since Salt reads all the top files and merges them together it meant frequent conflicts and errors.  This meant I would often have to update the top file in prod to test my changes in test or stage. Having to do this kind of workaround made my development process more complex and so I decided against that method.

Keeping an individual top file for each environment is simple to understand and gets rid of all conflicts in progressing changes through the environments.   But it does require breaking the git flow rules just a little.  top.sls is in the .gitignore file and is not in git.  In each of the pillar/state directories ( exmple: /data/repos/master/state, /data/repos/master/pillar) there is a symbolic link to the correct top file.

**pillar/ and state/ for the TEST environment**

```
top.sls.prod
top.sls.stage
top.sls.test
top.sls --> top.sls.test
```

**pillar/ and state/ for the STAGE environment**

```
top.sls.prod
top.sls.stage
top.sls.test
top.sls --> top.sls.stage
```

**pillar/ and state/ for the PROD environment**

```
top.sls.prod
top.sls.stage
top.sls.test
top.sls --> top.sls.prod

```

And the top files looks like this.

**pillar/top.sls for the TEST**

```
test:
    '*-*-test-*':
        - roles/common
        - parse_name
```

**pillar/top.sls for the STAGE**

```
stage:
    '*-*-stage-*':
        - roles/common
        - parse_name
```

**pillar/top.sls for the PROD**

```
base:
    '*-*-prod-*':
        - roles/common
        - parse_name
```

 Each environment on the server points to its own top file, and that top file maps the appropriate servers to that environment.  The first line of the top file is the environment the servers will be in and the second line is the match pattern.  Any server following the naming scheme will match one and only one of the above patterns and therefore is only in the environment specified.

 <sup>**Note**: At first I also had a catch all '\*-\*-\*-\*': in the base environment, but once I had all my servers migrated to this well-formed naming scheme I got rid of that.<br>


This takes care of getting the servers mapped to the environments and to have a workflow for development & testing before a salt change gets to PROD.  Next post I will talk about how to handle data that is specific to a given environment.
