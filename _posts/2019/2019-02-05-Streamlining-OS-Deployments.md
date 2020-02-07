---
title: "Streamlining OS Deployments"
date: "2019-02-05 23:55:00"
categories:
  - Automation
tags:
  - Ansible
  - Packer
---

> NOTE: Nothing in this post is earth shattering or bleeding edge. But, I do
> find that this is a very common conversation.

> ASSUMPTION: The following is based on vSphere, but can easily be adapted
> for any other virtualization platform.

## Background

How often do you find yourself maintaining OS images and/or templates? As most
of us know, this is a very tedious process that never ends. Spinning up a bare
VM, installing the OS, installing updates, adding a few base apps, etc. Once
this has been done, shutting down the VM and converting to a template. I'll
assume, that this sounds extremely familiar to many. I mean this is the way we
have been doing things for many years.

So, once you have your **GOLDEN** image/template, you are ready to spin up some
VMs by cloning this template. Now comes the fun part, keeping all existing VMs
in sync with your **GOLDEN** image/template. Assuming, that you have recently
updated this template for whatever reason. Do you approach this as the typical
**PET** vs. **CATTLE** scenario? Sorry, I had to go there. But, reality is, that
in most cases the **PET** scenario is the logical approach that most of us will
take. But why? Of course in many cases, this is a perfectly valid option, but
equally in as many other cases, this may not be a valid option. But, I digress.

## Enter Packer

[Packer](https://packer.io/) has been around for 6+ years now. So, this is not
a new kid on the block, or the latest hotness. Most of us, in some shape or form,
have heard of Packer. But, maybe not ever used it. I personally, have been using
Packer continuously for close to 4 years now. Packer is a lifesaver when it
comes to OS image building in a repeatable manner. But, the real beauty of Packer
is the ability to treat your images as code. Because everything Packer builds
from, **IS CODE**.

Packer provides the ability to consume an OS ISO, build the OS, execute any
provisioning scripts that you desire, and when complete, generate a consumable
image which can then be imported as VM template. Remember though, keep these
images to a bare minimum. Only installing the essential apps required, to further
hand off to your configuration management tool(s). Thinking in layers, rather
than cramming everything into a single image, ultimately resulting in numerous
images for a single OS. One OS, one image!

How does this solve the **PET** vs. **CATTLE** scenario? Well, it really doesn't.
However, it does begin to lay the foundation to approach the **CATTLE** scenario
much more easily. **More on this later!**

A major benefit to using Packer, is the fact that no one needs to remember
what and/or how was the last image built. Because everything Packer consumes, is
**IN THE CODE**. So yes, keep those Packer builds in a Git repository. So, each
new build, is just an iteration of the previous build(s).

## Enter Ansible

Like Packer, [Ansible](https://www.ansible.com/) has been around for 6+ years
as well. So, again, not a new kid on the block or the latest hotness. As most of
you know, I have been a huge proponent of Ansible for over 4 years. Ansible is
an example of a configuration management tool, as mentioned above, in regard to
handing off from Packer. Keep in mind, Ansible is not the only option for this,
but I am using it as the example.

So, how does Ansible fit into the picture here? First off, Ansible can be
leveraged to inject the images created by Packer into your environment. What
does this mean? Packer has the ability to export an image as a template directly
into a vSphere environment, but that is not what we are working towards here. We
want Packer to only export our image as an **OVA**, and then have Ansible import
that **OVA** into vSphere, and then convert it to a template. You may be wondering
why not just let Packer handle this process if it already supports it. I prefer
Packer to do it's job at generating the image, and that only.

> NOTE: Packer can absolutely be used to export your images directly as a
> template into vSphere, but there are requirements that must be met.
