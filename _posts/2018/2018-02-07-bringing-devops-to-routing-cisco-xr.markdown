---
title: "Bringing DevOps To Routing - Cisco XR"
date: "2018-02-07 21:54"
---

Is Cisco bringing DevOps to routing? Well that is what Cisco said while
presenting at [NFD17](http://techfieldday.com/event/nfd17/).

In the following video [Akshat Sharma](https://twitter.com/irakshat)
did a great job presenting their `Journey to the web`.

{% include video id="253197077" provider="vimeo" %}

> NOTE: I will be going rather deep into this platform as I really like where
> they are going with this. I also see a few interesting use cases which I will
> touch on towards the end of this post.

## What is Cisco XR

Cisco XR is based on the [Wind River Linux OS](https://www.windriver.com/products/linux/)
which brings the capabilities to run applications (native or [LXC](https://linuxcontainers.org/lxc/introduction/)/[Docker](https://www.docker.com/)), use
configuration management tools (Ansible, Chef, Puppet, and etc.), as well as
use many Open Source tools. This platform also allows for monitoring and
telemetry tooling to be used to handle operational aspects. Due to the push from
the web guys (Google, Facebook, Microsoft, and etc.) they moved from QNX (32-bit)
to 64-bit Linux in 2014. The ask was to include the ability to provide operational
focused solutions to be available within the platform. Cisco XR is also
multi-tenancy aware.

## Application Hosting

Cisco XR natively runs two primary containers which are considered to be system
containers. One being Cisco IOS XR (Control Plane) and the other being the Admin
Plane. The OS itself runs as a HyperVisor (libvirtd), and the Docker daemon
provides the container platform. There is also a third container space
(Third Party) which allows customers to run their own containers in that space.
This architecture builds along the lines of a container based modular system.
You also have the ability to run native applications within the Control Plane.
These applications resources are shared with internal IOS XR processes.

### Native Application Hosting Architecture

The diagram below lays out the architecture behind native application hosting.

![Cisco XR Native Application Hosting Architecture](images/2018/02/cisco-xr-native-application-hosting-architecture.png)

The `global-vrf` network namespace contains all of the native applications. This
namespace contains the following default routes:

-   Default route to XR FIB
-   Management routes

### Container Application Hosting

The applications ran in this space are completely isolated from IOS XR Control
Plane processes. There are two components within the this space:

-   Linux server - Used to develop applications, bring up Linux Containers (LXC),
    and prepare the container environment.
-   Router - 64-Bit IOS XR used to host containers.

### Docker Architecture

The diagram below lays out the architecture behind the Docker architecture on
IOS XR.

![Cisco XR Docker Architecture](images/2018/02/cisco-xr-docker-architecture.png)

### Docker Application Workflow

The ability to run Docker containers is an option within this platform. The
workflow to do such is as below:

-   Create a Docker image
-   Pull the image down using the Docker client in the XR Control Plane
-   Spin up the Docker container within the XR Linux shell

### Additional Application Hosting Resources

The cool thing (which I have not tested yet) is that you can also mock up scenarios
using Vagrant and IOS XR. You can find more info on [ios-xr/vagrant-xrdocs](https://github.com/ios-xr/vagrant-xrdocs)
and [xrdocs.github.io](https://xrdocs.github.io/). They also have a few additional
GitHub repos which can be found [here](https://github.com/ios-xr).

For a deeper dive into Application Hosting head over to Cisco's [Hosting Applications on IOS XR](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5000/app-hosting/b-application-hosting-configuration-guide-ncs5000/b-application-hosting-configuration-guide-ncs5000_chapter_011.html).

## ZTP/iPXE

IOS XR supports ZTP and iPXE which allows for automation and flexibility.

In the following video [Patrick Warichet](https://twitter.com/pwariche) explains
the ZTP/iPXE support in IOS XR.

{% include video id="253197037" provider="vimeo" %}

### ZTP

IOS XR supports ZTP(Zero Touch Provisioning) which allows the box to be provisioned
by user-defined scripts(bash/python) or a configuration file without any manual
intervention.

The diagram below illustrates the ZTP process:
![Cisco XR ZTP](images/2018/02/cisco-xr-ztp.png)

You can find more details on IOS XR's ZTP functionality [here](https://xrdocs.github.io/software-management/tutorials/2016-08-26-working-with-ztp/).

### iPXE

IOS XR also supports iPXE which provides the following benefits to the platform:

-   Boot via HTTP
-   Control boot process with scripts
-   control boot process with menus
-   DNS support
-   Chainloading

The diagram below illustrates the iPXE process:
![Cisco XR iPXE](images/2018/02/cisco-xr-ipxe.png)

## Final Thoughts

While participating in this session I found myself extremely interested in what
the platform provides. Especially as I had honestly never seen it before.
Understanding that I was not looking at this platform as purely a network device
but rather a TOR based network device that could potentially be used for provisioning
out racks of gear. Thinking of Webscale such as a massive container platform (completely automated).
This device could be provisioned via ZTP/iPXE, then spin up containers to provide
additional provisioning tools to further automate servers, switches, storage, and
etc. I made a reference to an appliance in the session and what I meant by that is,
using this device as my provisioning device for racks, datacenters, and etc. If
I have this device provisioned via ZTP/iPXE and have the ability to run configuration
management tools against it, I can then further use the same tooling to provision
out my whole infrastructure.

I plan on digging into this platform more and hopefully get my hands on either
the Vagrant environment or physical gear to further explore it's possibilities.
Being that it is Linux, I am sure that I can put this platform to good use!

> NOTE: You might think I have done this before or possibly built something similar! :)
>
> DISCLAIMER: I have been invited to Network Field Day 17 by Gestalt IT who
> paid for travel, hotel, meals and transportation. I did not receive
> any compensation to attend NFD and I am under no obligation whatsover
> to write any content related to NFD. The contents of these blog posts
> represent my personal opinions about the products and solutions
> presented during NFD17.
