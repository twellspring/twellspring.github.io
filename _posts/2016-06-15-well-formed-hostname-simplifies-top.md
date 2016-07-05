---
layout: post
title: Saltstack and a well formed hostname simplifies your top
tags: saltstack
---

###  The Problem ###
When I first started out using SaltStack, I setup a role based configuration where each server has one or more roles and those roles contain everything required to setup the server.  There is the **common** role which as you might expect from its name is applied to all server.  Then there will be at least one functional role.  My two most common are php and node.

A problem that quickly surfaced was how to connect a server ( minion ) to the pillars and states it needed.   Using the **top file** is the obvious answer. At its simplest the top file matches by minion ID, which in most instances is the hostname. So assuming my server names include php and node, the following top file works to configure the servers and can be used for both the pillar and the state top file.

**top.sls**

```
base:
    '*':
        - roles/common
    '*-php-*':
        - roles/php
    '*-node-*':
        - roles/node
```
**Update**: the top.sls paths can use dots in place of slashes if desired ( roles.common = roles/common), but I prefer the slashes.  This is from SaltStack's early decision to use the python import model. (Thanks for the reminder [@n_vogel](https://twitter.com/@n_vogel) )

But as I add more roles the top file gets more and more complex.   Right now I have 17 different roles.   So that is 17 different stanzas in the top file for my base environment.  And if I add in multiple environments ( test, stage, prod ) and servers with different functions that use the same node role, the top file is going to get really big and ugly.

###  Help from the Well Formed Hostname###

Using custom grains to identify role is proposed by some people, but that adds to the top file ugliness and has security implications.  On a compromised server the custom grains in /etc/salt/grains could be changed resulting in additional pillar data being exposed to the compromiser.  So to me that puts using custom grains on the **no way** list.

One piece of data from the minion that can not be altered is the ID, since that is part of the key that is accepted when salt minion is first installed.   Changing the ID means the key no longer matches and the minion can no longer connect to the master.  And since the ID is the hostname, we can use the hostname as a safe method for making decisions about a server.

Enter the well formed hostname.  Some people enjoy **creative** naming schemes: movie names, Game of Thrones characters, Greek gods.   Thats fine if they want to use them, but for me a well formed name gives much more of practical benefit.  I think at a minimum the well formed name needs:

* datacenter - I assume every organization has or will have more than one
* environment - test, staging, prod
* function - what the server is used for
* number - a way to distinguish multiple servers with similar functions apart

So the naming scheme for this example will be  datacenter-function-environment-number ( example: west-api-prod-1 ).  In this scheme the function will be used to map a server to a role and the corresponding pillars/states.  Function is not the clearest identifier so instead I am going to call it the nameslug.

###  Pillar Configuration ###
Lets put this nameslug to use to simplify the pillar top file.  Here is a simplified top file and a parse_name.sls file.

**pillar/top.sls**

```
base:
    '*-*-*-*':
        - roles/common
        - parse_name
```

**pillar/parse_name.sls**

```
{% raw %}
{%- set name_parts = {} %}
{%- set namesplit   = grains['id'].split('-') %}
{%- set nameslug = namesplit[1] }

include:
  - nameslugs/{{ nameslug }}
{% endraw %}
```

The top file includes parse_name which extracts the nameslug out of the ID/hostname and then includes a file in the nameslugs directory.  (nameslugs directory ... guess we need to create that.)   Each server type will have a corresponding "file" in the nameslugs directory.   While I could use actual sls files here, I found that using symbolic links in the nameslugs directory allowed it to be a more flexible mapping rather than holding content.  Where does it map to?  that depends on the server function.   Here is a simple pillar hierarchy

```
pillar
  nameslugs
  roles
    php
    node
    vpn
```

For a VPN server we would create a nameslug that maps to the vpn role.

```
cd nameslugs
ln -s ../roles/vpn vpn
```

The symbolic link targets the vpn folder.  In that folder is an init.sls file that contains the actual vpn role. An alternative would be to use vpn.sls as the file and have the symbolic link target vpn.sls.  But I have found that using a folder gives more flexibility to associate additional files with a given pillar/role/state.

And the vpn role is completed with the init.sls file.
** pillar/roles/vpn/init.sls **

```
roles:
  php: true

vpn:
  port:  1194
  ...

```

If I name a vpn server west-vpn-prod-1 it will be mapped to the correct role/pillars/states.  Goal accomplished for this one.  But if we have both a web server ( west-www-prod-1 ) and an api server ( west-api-prod-1 ) that need the PHP role we can not map directly between the nameslug and a role as there needs to be a way to differentiate the servers ( install the correct application ).  I will create a new **apps** directory to help resolve this.

```
pillar
  apps
    api
    www
  nameslugs
  roles
    php
    node
    vpn
```

Now the nameslugs for api and www map to the corresponding apps;

```
cd nameslugs
ln -s ../apps/api api
ln -s ../apps/www www
```

And the sls files in the apps hierarchy are used to add roles to these servers.  Here is the api file.

**pillar/apps/api/init.sls**

```
include:
  - roles/php

application:
  name: api
  git: git@github.com:mycompany/api.git
  ...
```

The api server west-api-prod-1 now has both been mapped to the php role and has the application specific data needed to install the correct application.

###  State Configuration ###

All of the above only gets us the correct pillar.  We need to also get the correct states on this server, and we use the roles pillar to do this.

**pillar/roles/php/init.sls**

```
php:
  version: 5.5
  user: www-data
  deploy_dir: /home/www-data/php

roles:
  php: true
```

**pillar/roles/common/init.sls**

```
common:
  packages:
    - git
    - nmap
    - top
    - zip

roles:
  common: true
```

Each rolls pillar contains both role specific data and then the general roles pillar.  Due to how the include logic of Saltstack works, if a server has multiple roles these roles are appended to the roles pillar.  Now in the top.sls I create a loop to on the roles pillar to include the state roles:

**state/top.sls**

```
{% raw %}
'*-*-*-*-*':
  {%- for role, args in salt['pillar.get']('roles', {}).items() %}
      - roles/{{ role }}
  {%- endfor %}
{% endraw %}
```

Based on the above a server named west-api-prod-1 would include the state roles common and api.

###  Summary ###

* The pillar top file includes parse_name
* parse_name.sls extracts the nameslug out of the server name and includes the corresponding nameslugs/xxxx
* nameslugs/xxxx is a symbolic link to either a role or an application folder.
* If it maps to an application, that application includes a role and creates an app specific pillar
* role files include a role specific pillar and an entry in the roles pillar
* state top file includes all roles from the roles pillar via a jinja loop

This structure has worked well for me.  I started out with 2 roles and 3 apps and now with 17 roles and 35 apps it still works.  Instead of a long complex top file, I have a well organized hierarchy with lots of small easy to understand files with well defined content.

So what about the other parts of the **well formed hostname**? Those parts are useful and will be well used, but that is for another post.
