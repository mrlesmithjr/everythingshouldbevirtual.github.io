---
title: "Cumulus - Not Your Average Networking"
date: "2018-03-19 16:54"
categories:
  - Networking
tags:
  - Cumulus
---

## Network Field Day 17

[Pete Lumbis](https://twitter.com/PeteCCDE) did an amazing job presenting [Cumulus](https://cumulusnetworks.com/)
at [#NFD17](https://twitter.com/hashtag/NFD17?src=hash) and I highly recommend
you check it out.

{% include video id="253197258" provider="vimeo" %}

## Adoption

Cumulus is nothing new to the networking world as they have been doing
some very cool things over the past several years. However, I feel that we are
seeing their adoption more prevelant than before. There are several reasons for
this in my belief. Foremost I feel the main driver for this adoption is that more
and more businesses are willing to embrace non big vendor networking because of
a multitude of reasons. And yet another reason is that more and more networking
engineers are becoming more familiar with the Linux CLI and automation.

## Open Source

For those of you not familiar with Cumulus, Cumulus is based on Debian Linux from
a core OS perspective. They obviously have some super secret sauce that they lay
down but for the most part they utilize various Open Source solutions. One of which
originated from Quagga is [Free Range Routing (FRR)](https://frrouting.org/). Now
if you have never had the experience of working with Quagga or FRR you are definitely
missing out. I have had the privilege of working with Quagga in large scale multi-datacenter
deployments using BGP. And I must say it works very very well.

## CLI

Now what Cumulus has done from a CLI perspective is a thing of beauty. They have
accounted for traditional network engineers to still use familiar CLI commands
which translate into Linux primitives without them needing to understand Linux
CLI commands. But at the same time, they leave the traditional Linux environment
as is allowing for those familiar with Linux to manage the OS as they normally
would.

For more info on Cumulus CLI utility head over to [Network command line utility (NCLU)](https://docs.cumulusnetworks.com/display/DOCS/Network+Command+Line+Utility+-+NCLU).

![Cumulus NCLU](../../images/2018/03/cumulus-nclu.png)

## Automation

When it comes to automation, because Cumulus is just a Debian Linux OS, we can
easily automate the switch using any of our preferred configuration management
tools. Because of this, we are not at the liberty of say an Ansible module to be
developed in order for us to manage a Cumulus switch. This is very powerful to
say the least. The other aspect around automation is that Cumulus provides [Cumulus VX](https://cumulusnetworks.com/products/cumulus-vx/)
which is a virtual appliance which can be spun up in a multitude of ways. With
this ability available to us, we can easily mock up deployments and get our
automation developed and tested prior to configuring our production environment.

### Ansible

If you are an Ansible user, I highly recommend you checkout my existing Ansible
roles below:

-   [ansible-quagga](https://github.com/mrlesmithjr/ansible-quagga)
-   [ansible-frr](https://github.com/mrlesmithjr/ansible-frr)

## Final Thoughts

I have only touched on a few points in this post that makes Cumulus a compelling
solution but I can ensure you that they are for real. I highly recommend you give
them a look and see for yourself just how cool their solution really is.

> NOTE: You might think I have done this before or possibly built something similar! :)

I plan on digging into this platform more and hopefully get my hands on either
the Vagrant environment or physical gear to further explore it's possibilities.
Being that it is Linux, I am sure that I can put this platform to good use!

> DISCLAIMER: I have been invited to Network Field Day 17 by Gestalt IT who
> paid for travel, hotel, meals and transportation. I did not receive
> any compensation to attend NFD and I am under no obligation whatsover
> to write any content related to NFD. The contents of these blog posts
> represent my personal opinions about the products and solutions
> presented during NFD17.
