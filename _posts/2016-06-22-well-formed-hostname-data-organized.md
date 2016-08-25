---
layout: post
title: Saltstack and a well formed hostname keeps your environment data organized
teaser: "Previously the well formed hostname has been used to: 1) map servers to roles
2) map application servers sharing roles to the appropriate role and to the application specific data needed to customize each server 3) map servers to salt environments"
tags: saltstack
---

Previously the well formed hostname has been used to

* map servers to roles
* map application servers sharing roles to the appropriate role and to the application specific data needed to customize each server.
* map servers to salt environments

Similar to applications needing app-specific data, using environments require the addition of environment specific data.  Here are some of the data that is different for me between stage and prod

* Database server names & credentials
* SSL certificates
* User accounts
* Test/PROD Logins for external web services
* Credentials used by web servers to authenticate with application servers

###  Environment Files ###
The pillar for a given role or application needs environment specific data like the above and data that is not environment specific  ( app name, install directory, .... ).  I want a way to make it clear what data belongs to what environment.   To do this, lets start with the pillar from a couple of posts ago, but add in some files.

```
apps
  api
    init.sls
    prod.sls
  www
nameslugs
roles
  php
    init.sls
    stage.sls
    prod.sls
  node
  vpn
```

I have expanded one application and one role to include the default file ( init.sls ) and files named after the environment.   Anything in the default file applies to all environments.  Anything in the environment named file applies only to that environment.

This makes things clear in the filesystem, but how do we get it to work in salt?  For instance west-api-prod-1 would need to include the files apps/api/init.sls and apps/api/prod.sls.  But west-api-stage-1 will only include apps/api/init.sls since apps/api/stage.sls does not exist.   If a staging server tries to include the non-existent apps/api/stage.sls it will result in an error.  There is no ( at least that I can see ) way to specify this by hand ... there has to be a programatic way to look at the environment and include the relevant environment specific file, but only if it exists.  The name for this is **optional include**  and Saltstack has some optional includes functionality, but it is not complete yet ( does not work everywhere), so we need another solution.   Enter Jinja.

###  Jinja Optional Include ###

Jinja is the python template language and is available in SaltStack.  Jinja can be used for many things in salt pillars, states and template files.  In this case it can be used to create an **optional_include** macro ( a re-usable piece of code).   I got the idea for this macro from an issue [SaltStack Issues board on github](https://github.com/saltstack/salt/issues) a couple of years ago and would give credit if I could remember where I found it and the issue is long since closed.  Here is the code:

**pillar/macro.jinja**

{% raw %}
```
{%- macro include_optional(sls_file) %}
  {%- for root in opts['pillar_roots'][saltenv] -%}
    {%- if salt['file.file_exists']('{0}/{1}.sls'.format(root, sls_file )  %}
    - {{ sls_file }}
    {% endif %}
  {%- endfor %}
{%- endmacro %}
```
{% endraw %}

This macro is passed a normal pillar include path.  It checks if the sls file exists.  If so, it adds the appropriate include line.  Following are the pretty dense technical details ( skip ahead to the next paragraph if you don't need to know how it works ).  The complexity here is that the **file.file_exists** jinja command requires an absolute path to an actual file, but the pillar include path being passed to the macro is a pillar name relative to the pillar_root.  To convert this relative path to an abolute, the macro uses **opts['pillar_roots'][saltenv]** to get a list of the absolute paths to each pillar_root in the current environment.  Since we have only one pillar root for each environment this returns a single value ( but the macro will work with multiple pillar_roots).  Using the jinja **format** function to combine text and variables, a string containing the absolute path to the SLS file is created.  Then **file.file_exists** checks if that file exists and adds an include line (relative to pillar_root and without the .sls) to the pillar file.  As long as the Jinja is interpreted before the salt pillar include logic, then the salt pillar gets included just like it was hard-coded.

A macro is used by importing it and then referencing it in the sls file.   Here is the api/init.sls file from last post with an optional include of the appropriate environment specific php file.

**pillar/apps/api/init.sls**

{% raw %}
```
#!jinja|yaml
{%- from "macros.jinja" import include_optional with context %}

include:
  - roles/php
  {{ include_optional( 'roles/php/{0}'.format( saltenv)  ) }}

application:
  name: api
  git: git@github.com:mycompany/api.git
  ...
```
{% endraw %}

The first line here is SaltStack's version of the [Linux shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)).  This line tells Salt which interpreters to run and in which order.  Jinja runs first so it processes the include_optional before the yaml (which also performs the **include** logic). For west-api-prod-1 the line

```
  - app/api/prod
```

would be added to the includes but for west-api-stage-1 there would be no addition as **app/api/stage.sls** does not exist.

###  Pillar Merging ###
Before leaving this topic I want to briefly delve into pillar merging.  Merging was used before this post, but knowing how it works is important so that there are no surprises when the global and environment specific files are combined. The [SaltStack Pillars documentation](https://docs.saltstack.com/en/latest/topics/pillar/) covers this but is a little confusing.  Lets simplify it: if you include two pillars, the one included last overwrites the previous if there is overlap.   What is overlap?  When both have the same key.

**First Pillar**

```
application:
  name: api
  monitoring: false
  requires:
    - file1
    - file2
```

**Second Pillar**

```
application:
  monitoring: true
  ssl: enabled
  requires:
    - file3
```

**Merged Pillar**

```
application:
  name: api
  monitoring: true
  ssl: enabled
  requires:
    - file1
    - file2
    - file3
```

Monitoring is an overlap as the same key (application: monitoring) exists in both, so the second value is used.  But it is not quite that simple when it comes to lists.   There are different behaviors that you can specify in the salt config, but the default behavior for a list would be to append the items.  So the list includes both the items from the first and second pillars.

This merge behavior allows the generic pillar ( init.sls )  to set a default value and then to have it overwritten by a more specific value in the environment specific pillar, as long as they are merged in the correct order.

This merging strategy will be expanded upon next post, along with using another part of the well formed name.
