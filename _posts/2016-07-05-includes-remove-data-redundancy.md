---
layout: post
title: Saltstack expanding includes to remove data redundancy
tags: saltstack
---

Last post I created a pillar merge strategy that allowed data at different scopes and include it on all the servers relevant to that scope. After implementing this strategy and using it for a while I started having the needing for the same data in multiple different pillars. Here are example pillars for two applications that need the same credentials for a web services like [segment.com](https://segment.com).

**apps/www.sls**

```
apps:
  www:
    envdata:
      web_service1_url:
      web_service1_login:
      web_service1_password:
```

**apps/customer_app.sls**

```
apps:
  cutomer_app:
    envdata:
      web_service1_url:
      web_service1_login:
      web_service1_password:
```

I want to define credentials like this in one place and include it where needed. But that does not work using the SaltStack pillar include. My pillar design with the application name at the second pillar level allows for multiple applications on a single server, but it also means the same data is in different pillar locations.

* apps:www:envdata:web_service1_login
* apps:customer_app:envdata:web_service1_login

Since sls file included via SaltStack needs to have the full pillar definition ( apps:xxxx:envdata) this means I can not use this kind of include.  Removing the app name from the app pillar would remove this issue, but would limit salt to one app per server.  I want to keep the flexibility of multiple apps on a server, so I went looking for another option.

First I thought of putting common data in a common location out of the application pillar ( say services:web_service1 ), but my states are designed to use the apps:appname pillar for all their configuration and having to get data from multiple locations makes states much harder to write and maintain.  Expanding on this idea, how about moving the data and then having a pointer.  Something like this:

```
include:
  - services
  - apps/customer_app
```

**services/init.sls**

```
services:
  web_service1:
   url:
   login:
   password:
```
**apps/customer_app.sls**

```
apps:
  cutomer_app:
    envdata:
      service: web_service1
```

Maybe a little better theoretically since the apps pillar can be used to get or get references for all the required data.   But once again the states are more complex because they have to inspect the items in the envdata pillar and make the substitutions. I want simplicity in the pillar and simplicity in the states. With normal pillar includes I can't find a solution, but with Jinja I can.  Enter stage right the Jinja **include**.

**apps/www/init.sls**

{% raw %}
```
#!jinja|yaml|gpg

apps:
  www:
    envdata:
{% include 'services/web_service1.sls' %}
```

**apps/customer_app/init.sls**

```
#!jinja|yaml|gpg

apps:
  cutomer_app:
    envdata:
{% include 'services/web_service1.sls' %}
```

**services/web_service1.sls**

```
      web_service1_url:
      web_service1_login:
      web_service1_password:
```
{% endraw %}

The jinja include adds the three lines from web_service1.sls right under envdata, effectively giving the same pillar as at the top of this blog.  Unlike the jinja **file.file_exists** the Jinja include takes a path relative to the pillar root.  But it does require the full file name, hence the .sls in the include lines.

An important point to note is the indenting in the web_service1.sls file. My initial experimenting with the jinja include showed that indenting the jinua include line did not mean that all of the included lines would be correctly indented. From what I remember the first line of the include was indented and the remaining lines were not.  That does not fly with yaml as indenting is important.  Not indenting the include line and putting the correct indents in the service file got the included lines to all be indented correctly.  As long as the indent level is consistent across all pillar files that include web_services1.sls, this solution works.  And since I have a well formed structure for my apps pillar, the indenting is always the same.

I like the organization and clarity this provides in my pillar file system.  All external services can be defined in one folder.  And jinja includes allow me to include this data on just the applications that need it. As I look through my application pillar files I find most of them now have several jinja includes for things like:

* SSL certificates ( this is really helpful for wildcard certs)
* web service credentials
* Internal API credentials
* Shared secrets
