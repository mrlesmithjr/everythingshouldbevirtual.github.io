---
title: "Ansible - Using YAML For Inventory"
date: "2018-04-23 23:40"
categories:
  - Automation
tags:
  - Ansible
---

## Backgroud

When it comes to Ansible inventory most of us are probably more familiar using
the standard INI method. And for those of us who have had the luxury of maintaing
these INI inventories in massive scale learn to really despise this format for
one reason or another. So in this post I will quickly show how we can use YAML
to define our inventory which will provide us a much cleaner view of our inventory
and quite honestly much easier to manage.

## The INI Inventory

As mentioned above most of us are probably the most familiar with this format.
But for reference we will look at what this might look like in INI format.

```bash
green.example.com ansible_host=10.0.101.100
blue.example.com
192.168.100.1
192.168.100.10

[webservers]
alpha.example.org
beta.example.org ansible_host=192.168.200.122
192.168.1.100
192.168.1.110
www[001:006].example.com

[webservers:vars]
nginx_http_port=80
nginx_https_port=443

[dbservers]
db01.intranet.mydomain.net
db02.intranet.mydomain.net
10.25.1.56
10.25.1.57
db-[99:101]-node.example.com
```

As you can see from the example above, we have a mix of hosts defined in a few
different ways. The first few are considered ungrouped, then we have our
grouped hosts, then we have also defined some variables specific to our webservers
group.

## The YAML Inventory

If we take the above example in INI format and convert that to a YAML inventory
we would end up with something that looks similar to:

```yaml
---
ungrouped:
  hosts:
    green.example.com:
      ansible_host:               10.0.101.100
    blue.example.com:
    192.168.100.1:
    192.168.100.10:

webservers:
  hosts:
    alpha.example.org:
    beta.example.org:
      ansible_host:               192.168.200.122
    192.168.1.100:
    192.168.1.110:
    www[001:006].example.com:
  vars:
    nginx_http_port:              80
    nginx_https_port:             443

dbservers:
  hosts:
    db01.intranet.mydomain.net:
    db02.intranet.mydomain.net:
    10.25.1.56:
    10.25.1.57:
    db-[99:101]-node.example.com:
```

And as you can see, defining our inventory in YAML has a much better flow and
is much easier to manage. Again, these are just examples of how our inventory
might look using YAML.

Enjoy!
