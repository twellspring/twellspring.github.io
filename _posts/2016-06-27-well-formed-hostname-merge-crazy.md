---
layout: post
title: SaltStack and the well formed hostname prevents merge chaos
tags: saltstack
---

The pillar merge used in the last post allowed us to have global data and environment specific data.   Now I want to add in another: datacenter specific data.   Adding a pilar merge for datacenter is not difficult requiring just two additional lines in the parse_name.sls.

**pillar/parse_name.sls**

```
{% raw %}
{%- set name_parts = {} %}
{%- set namesplit   = grains['id'].split('-') %}
{%- set datacenter = namesplit[0] }
{%- set nameslug = namesplit[1] }

include:
  - nameslugs/{{ nameslug }}
  - datacenter/{{ datacenter }}
{% endraw %}
```

## Pillar Merge Strategy ##
This small addition is probably not worth an entire blog post.  But it is the gateway introduce the need for a pillar merge strategy. The well formed hostname ( datacenter-function-environment-number ) makes it is possible to have data specific to the each name part and to the whole name:

* datacenter
* function
* environment
* number
* hostname

There are valid use cases for data specific to each of these, except for number specific data (any hostname with number 42 get a towel pillar???).  So lets start with this list ( minus number ) to create a full list of the specific types of data that we might need.   First I will add at the top a line for global data, which I call defaults.  Then I will replace function with application/role since that is what function gets mapped to via the nameslug.  So here is the updated list of types of pillar data:

* defaults
* datacenter
* environment
* role
* application
* hostname

I want to separate out different types of data into different files in my pillar filesystem.   These files are then brought together by includes, which per the previous blog means any data overlap will cause the first included item to be overwritten by the second. That means there needs to be a well planned merge order so that I always know what data the resulting pillar will contain.  Above is my theoretical merge order.  Below is my logic for this order.  Yours might be different, but that is fine as long as you understand the logic and how that affects the final pillar.

The first one is easy.  By definition defaults should be overwritten by any of the other data types in the list, so it will be merged first.  What is next?   Does datacenter specific data overwrite environment specific data or visa versa?  How about environment vs role?  I started my decision process by thinking about the use case for each category.  Datacenter specific data use case was information required for basic network functionality:

* NTP server
* Internet Gateway IP
* Mail gateways
* Private Ethernet Adapter ( example: AWS uses eth0, rackspace eth2 )

This information is not likely to be changed by any of the other categories so is relatively independent and could therefore either be before environment/function/hostname or after them.  I decided to put it before.

The use case for environment specific data here is general settings related to the environment.  Examples:

* Should monitoring be enabled
* What users have accounts
* Setting environment specific resources like db server

The use cases for datacenter and environment probably do not have any overlap. But there will be overlap between environment/role/application.  For instance in the prod environment we enable monitoring.  But I have a server role for running high impact batch jobs.  Having monitoring enabled will mean lots of false positives.  So this batch worker role either needs monitoring disabled or different thresholds for monitoring.  So what order makes sense for these last few items.

The merge ordering as I am seeing it is working on decreasing scope, and I like the idea of a merge order based on scope. Datacenter is the largest scope as it can include multiple environments/functions making it next in the list after defaults and overwriting any defaults.  In my thinking Environment's scope is smaller than datacenter as each datacenter can have multiple environments, but bigger than role/application/hostname.  So next comes environment which will overwrite any overlaps with datacenter.  To continue ordering by scope all the way down means after environment comes role, application, hostname.

## Pillar Merge Code ##
So now this theoretical merge order needs to turn into code.  Here is the actual merge order that the code will implement:

* defaults
* datacenter
* environment
* common role
* application
* application-environment
* role
* role--environment
* hostname

The include_optional code provided environment specific data within a particular application or role so I added in application-environment and role-environment.  And we already had the common role so I will add it in too.  Notice that I switched the application and role order due to a technical issue I will explain next after showing the code that implements this order.

**pillar/top.sls**

```
base:
    '*-*-prod-*':
        - parse_name
```

The only change from before in the top is to remove the roles/common.  It was moved into parse_name so it can be properly sequenced.

**pillar/parse_name.sls**

```
{% raw %}
{%- set name_parts = {} %}
{%- set namesplit   = grains['id'].split('-') %}
{%- set datacenter = namesplit[0] }
{%- set nameslug = namesplit[1] }
{%- set env = namesplit[2] }

include:
  - defaults
  - datacenter/{{ datacenter }}
  - env/{{ env }}
  - roles/common
  - nameslugs/{{ nameslug }}
{% endraw %}
```

<sub>Note: The environment can be specified using either the env split from the name and shown here or the **saltenv** variable.  Since **saltenv** is set in the top file based on the value in the third section of the name, these two are functionally equivalent.

The jinja now pulls out datacenter and env which are used in the new include lines.   This include list looks like the merge list, at least from roles/common up.  The bottom half of the include list is found in the file included with nameslugs, which per our example would be api.

**pillar/apps/api/init.sls**

{% raw %}
```
#!jinja|yaml
{%- from "macros.jinja" import include_optional with context %}

include:
  {{ include_optional( 'apps/api/{0}'.format( saltenv)  ) }}
  - roles/php
  {{ include_optional( 'roles/php/{0}'.format( saltenv)  ) }}


application:
  name: api
  git: git@github.com:mycompany/api.git
  ...
```
{% endraw %}

The pillar data in an sls file is overwritten by any pillars included in that file.  Since include pillars are merged in they order they appear in the include, the above code creates a merge order of:

* apps/api/init
* apps/api/prod
* roles/php
* roles/php/prod

## Merge Technical Problem ##
This is not the same order as my theoretical merge order of decreasing scope which has roles before apps (as to me role is larger in scope than an app).  But because the nameslug symbolic link points at the application sls file it is not possible to have roles before application.  Replacing the nameslug symlink with a file like this would allow the theoretical order to be kept:

**pillar/nameslug/api.sls**

{% raw %}
```
include:
  - roles/php
  {{ include_optional( 'roles/php/{0}'.format( saltenv)  ) }}
  - apps/api
  {{ include_optional( 'apps/api/{0}'.format( saltenv)  ) }}
```
{% endraw %}

But I don't like this because it makes the code more complex as the application is defined partially in the nameslug file and partially in the app file.  And so I decided this minor deviation from scoping was the more acceptable of these two options.  And as long as I have a well defined merge order I will know how the data in my apps and roles will merge.  The practicals of this re-ordering is that roles can not be over-written by an application so all applications have to use the functionality put in place by a role.   This does not seem like much of a problem.   And in practice I have not had any difficulty with this ordering.

I was about a month into my salt deployment when I realized how important having this well defined merge strategy was.   I think looking back now that this was one of the most important theories/code to have in place.   Every time I add new functionality like a new role or a new application type, having this well defined separation of data types and the well defined merge order makes the job of where to put pillar data straightforward.
