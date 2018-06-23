---
title: "Using ARA Records Ansible Within Projects"
date: "2018-06-22 23:10"
categories:
  - Automation
tags:
  - Ansible
  - Reporting
---

While exploring some options to provide a level of reporting capabilities as part
of a project I figured using [ARA](https://github.com/openstack/ara) would be
an awesome idea. My goal was to provide a way to include ARA as part of Git
Repositories(projects) in which the consumers of those projects had the ability
to gain visibility into reporting of Ansible executions. The idea would be to
have ARA use a folder within the project such as `logging/.ara` which would become
part of the project which could be committed for historical reasons. Which then
could be served up from anywhere once the components of ARA are committed.

Sounds cool right? Well at least to me it seems to make a lot of sense. But as I started
looking at the options of using ARA and configuring my `ansible.cfg` file which
is within each project already I soon realized for some reason this would not
work as I expected. So if we look at what the `ansible.cfg` file might look like
as what I would imagine would work:

`ansible.cfg`

```bash
[defaults]
callback_plugins=/usr/local/lib/python2.7/site-packages/ara/plugins/callbacks
action_plugins=/usr/local/lib/python2.7/site-packages/ara/plugins/actions
library=/usr/local/lib/python2.7/site-packages/ara/plugins/modules

[ara]
dir=./.ara
```

> NOTE: The above information under defaults was obtained by running the following
> command: `python -m ara.setup.ansible`.

So what I found from the above is that ultimately ARA does not like for some
reason defining `dir=./.ara` but I could use the full working directory and it
was happy. Well that is not going to work as I will never know exactly where a
consumer of these projects might actually be running Ansible from. In addition
to this, the settings under `[defaults]` would most likely be different in each
and every scenario as well. What if I put this under CI/CD? Who knows what that
would look like and it surely could be different each and every time as well.

So how did I finally solve this? In all reality it is rather quite simple. And
it will definitely work out much better IMHO as maybe the consumer may not want
to always generate some ARA reporting output from their Ansible executions. So,
to solve this, I would just simply add the following shell script as part of
each project and let the consumer decide whether or not they wanted to generate
ARA reporting. With that being said, the following is all it would take to make
this achievable.

`ara_setup_env.sh`:

```bash
#!/usr/bin/env bash

ARA_DIR=$PWD/logging/.ara
ANSIBLE_CALLBACK_PLUGINS=$(python -m ara.setup.callback_plugins)
ANSIBLE_ACTION_PLUGINS=$(python -m ara.setup.action_plugins)
ANSIBLE_LIBRARY=$(python -m ara.setup.library)

export ARA_DIR
export ANSIBLE_CALLBACK_PLUGINS
export ANSIBLE_ACTION_PLUGINS
export ANSIBLE_LIBRARY
```

Now when a consumer would like to generate ARA reporting they can simply just
execute the following `source ara_setup_env.sh` before executing their Ansible
playbook. ARA would then use the current working directory and `logging/.ara` and
then dynamically define `ANSIBLE_CALLBACK_PLUGINS`, `ANSIBLE_ACTION_PLUGINS`,
and `ANSIBLE_LIBRARY` no matter which environment ran from assuming that ARA has
been installed.

This would also play nicely when running through CI/CD processes as well so this
should solve the requirement that I was aiming for!

Hope you enjoy and maybe someone else might find some use of this.

Enjoy!
