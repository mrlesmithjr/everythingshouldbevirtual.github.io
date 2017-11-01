---
  title: Docker - Creating An Alpine Image With Ansible
---

As I am experimenting with [Docker](https://www.docker.com) images and finding
the right combination which works well for me the majority of the time. I have
finally put together a simple `Dockerfile` which uses [Alpine](http://alpinelinux.org)
Linux as the base image and installs [Ansible](https://www.ansible.com). Why
[Ansible](https://www.ansible.com/) in a [Docker](https://www.docker.com) image?
Because for me it allows me to get around `BASH` and use a desired state
methodology. Now this may not be for everyone but it works really well for me.
Especially when I am using this base image to do additional complex configuration
either during the image creation or during the spin-up of a container which may
require custom provisioning. So, with this all being said you will find the very
simple `Dockerfile` below which will install [Ansible](https://www.ansible.com/)
for us.

`Dockerfile`

```bash
FROM alpine:3.4

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

RUN apk update && \
apk add --no-cache ansible && \
rm -rf /tmp/* && \
rm -rf /var/cache/apk/*
```

To build your image simply create the `Dockerfile` above and then build the image:

```bash
docker build -t alpine-ansible .
```

So why [Alpine](http://alpinelinux.org/) Linux? Because it is small... How small?
Let's compare a few base images.

`Ubuntu 14.04`

```bash
ubuntu 14.04 b1719e1db756 34 hours ago 188 MB
```

`Ubuntu 16.04`

```bash
ubuntu 16.04 45bc58500fa3 34 hours ago 126.9 MB
```

`Debian Jessie`

```bash
debian jessie a24c3183e910 33 hours ago 123 MB
```

`Alpine 3.4`

```bash
alpine 3.4 7d23b3ca3463 28 hours ago 4.799 MB
```

Now let's look at what a [Ubuntu](http://www.ubuntu.com/) image looks like with
[Ansible](https://www.ansible.com/) installed:

```bash
mrlesmithjr/ubuntu-ansible latest 69c2195ca7d7 31 hours ago 242.2 MB
```

As you can see, installing [Ansible](https://www.ansible.com/) added about
`~120MB` to our base image. Now let's see what an [Alpine](http://alpinelinux.org/)
image looks like with [Ansible](https://www.ansible.com/) installed:

```bash
mrlesmithjr/alpine-ansible latest 5291ba47263e 26 hours ago 65.01 MB
```

Wow...We have only increased our [Alpine](http://alpinelinux.org/) base image
by `~60MB`... This cuts out almost half of the size of even a `Debian` or
`Ubuntu` base image.

So there you have it...

Now you can leverage my pre-build [Alpine](http://alpinelinux.org/) image with
[Ansible](https://www.ansible.com/) installed in a new `Dockerfile` to create a
new image for another app:

`Dockerfile`

```bash
FROM mrlesmithjr/alpine-ansible

...
```

Or, even build your own image now.

I will be creating another post in the near future on how to build additional
images for apps which will leverage this [Alpine](http://alpinelinux.org/) image
with [Ansible](https://www.ansible.com/).

Enjoy!
