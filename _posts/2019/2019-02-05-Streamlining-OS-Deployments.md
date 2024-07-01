---
title: "Streamlining OS Deployments"
date: "2019-02-05 23:55:00"
categories:
  - Automation
tags:
  - Ansible
  - Packer
---
> UPDATE: **2023-05-11** - I found this post in my backlog, and it was never published or finished, as you can see. This conversation continues to be something that I regularly have. So, I will leave this post unfinished and eventually publish an updated version or an appendix.

> NOTE: Nothing in this post is earth-shattering or bleeding-edge, but this is a very common conversation.

> ASSUMPTION: The following is based on vSphere but can easily be adapted for any other virtualization platform.

## Background

How often do you find yourself maintaining OS images or templates? As most know, this is a tedious process that never ends—spinning up a bare VM, installing the OS, installing updates, adding a few base apps, etc. Once this has been done, shut down the VM and converting to a template. I'll assume that this sounds extremely familiar to many. This is how we have been doing things for many years.

So, once you have your **GOLDEN** image/template, you are ready to spin up some VMs by cloning this template. Now comes the fun part: keeping all existing VMs in sync with your **GOLDEN** image/template and assuming you have recently updated this template for whatever reason. Do you approach this as the typical **PET** vs. **CATTLE** scenario? Sorry, I had to go there. But, the reality is that, in most cases, the **PET** scenario is the logical approach that most of us will take. But why? Of course, this is a perfectly valid option in many cases, but equally, in many other cases, this may not be a good option. But I digress.

## Enter Packer

[Packer](https://packer.io/) has been around for 6+ years now. So, this is not a new kid on the block or the latest hotness. Most of us, in some shape or form, have heard of Packer. But maybe it has yet to be used. I have been using Packer continuously for close to 4 years now. Packer is a lifesaver in terms of OS image building in a repeatable manner. However, the real beauty of Packer is the ability to treat your images as code. Because everything Packer builds from **IS CODE**.

Packer provides the ability to consume an OS ISO, build the OS, execute any provisioning scripts that you desire, and, when complete, generate a consumable image, which can then be imported as a VM template. Remember, though, to keep these images to a bare minimum. Only installing the essential apps is required to further hand them off to your configuration management tool(s). Thinking in layers, rather than cramming everything into a single image, ultimately resulting in numerous images for a single OS. One OS, one image!

How does this solve the **PET** vs. **CATTLE** scenario? Well, it doesn't. However, it does begin to lay the foundation to approach the **CATTLE** scenario much more quickly. **More on this later!**

A significant benefit to using Packer is that no one needs to remember what and how the last image was built. Because everything Packer consumes is **IN THE CODE**. So yes, keep those Packer builds in a Git repository. So, each new build is just an iteration of the previous build(s).

## Enter Ansible

Like Packer, [Ansible](https://www.ansible.com/) has been around for 6+ years as well. So, again, I'm not a new kid on the block or the latest hotness. As most of you know, I have been a massive proponent of Ansible for over four years. As mentioned above, Ansible is an example of a configuration management tool regarding handing off from Packer. Remember, Ansible is not the only option for this, but I am using it as an example.

So, how does Ansible fit into the picture here? First, Ansible can be leveraged to inject the images created by Packer into your environment. What does this mean? Packer can export an image as a template directly into a vSphere environment, but we are working towards something else. We want Packer only to export our image as an **OVA**, have Ansible import it into vSphere, and then convert it to a template. Why not let Packer handle this process if it already supports it? I prefer Packer to do its job of generating the image, and that only.

> NOTE: Packer can be used to export your images directly as a template into vSphere, but some requirements must be met.
