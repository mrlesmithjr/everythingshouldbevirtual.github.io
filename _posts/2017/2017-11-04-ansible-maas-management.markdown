---
title: "Ansible - MAAS Management"
date: "2017-11-04 21:48"
categories:
  - Automation
tags:
  - Ansible
  - MAAS
---

Several months back I was working quite a bit with [MAAS](https://maas.io/) and
needed a way to fully automate not only the MAAS deployment but also the
management of VMs. In this case they were VMs running on [KVM](https://www.linux-kvm.org).

So as I am backfilling with posts and catching up on some things that I had
intended on sharing this would be one of them.

First thing is to grab the [ansible-maas](https://github.com/mrlesmithjr/ansible-maas)
Ansible role if you would like to use that. This role is actually a very basic
single host deployment but can easily be adapted to separate out functionality
if needed.

Once you have your MAAS deployment completed you are then ready for managing MAAS
with what else other than, **Ansible** of course. Below you will find the templates
and playbook used for managing MAAS.

To generate MAAS Ansible inventory you can use the following Jinja2 template:
{% gist mrlesmithjr/2051ac365967a8450e838894160f8a94 %}

To generate MAAS related variables based dynamic discovery you can use the
following Jinja2 template:
{% gist mrlesmithjr/bb90e571250aa31e2d39dad70383b895 %}

And for the actual MAAS management playbook you can use the following:
{% gist mrlesmithjr/05ef3e7c996d3358f939f7e6542cd647 %}

Of course there is so much more you can do with this but I really just wanted to
share the foundation of the MAAS management here for others if there is ever a
need.

Enjoy!
