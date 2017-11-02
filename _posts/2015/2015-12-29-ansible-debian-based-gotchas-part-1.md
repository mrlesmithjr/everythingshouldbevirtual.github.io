---
  title: Ansible - Debian Based Gotchas - Part-1
---

As I am currently running through some Docker setups between Debian
Jessie and Ubuntu Trusty using Ansible I ran into a few gotchas. Now I
would have assumed that everything SHOULD be identical but not so much.
With this being said, I figured I would begin to start putting some
context around some gotchas that I run into from time to time. And
hopefully this will help out others as well.

## Docker

When using the following task which is part of this Ansible role for
Docker.
[ansible-docker](https://github.com/mrlesmithjr/ansible-docker)

{% raw %}

```yaml
---
- name: debian | updating apt-cache
  apt:
    update_cache: yes
    cache_valid_time: 86400
  become: true

- name: debian | installing pre-reqs
  apt:
    name: "{{ item }}"
    state: present
  become: true
  with_items:
    - 'apt-transport-https'
    - 'ca-certificates'
    - 'software-properties-common'

# We are removing the old Docker info
- name: debian | Removing Legacy Docker apt-key
  apt_key:
    keyserver: "hkp://p80.pool.sks-keyservers.net:80"
    id: "58118E89F3A912897C070ADBF76221572C52609D"
    state: "absent"
  become: true

# We are removing the old Docker info
- name: debian | Removing Legacy Docker Repo
  apt_repository:
    repo: "deb https://apt.dockerproject.org/repo {{ ansible_distribution | lower }}-{{ ansible_distribution_release }} main"
    state: "absent"
  become: true

- name: debian | adding docker apt-key
  apt_key:
    url: "{{ docker_ubuntu_repo_info['url'] }}"
    id: "{{ docker_ubuntu_repo_info['id'] }}"
    state: "present"
  become: true

- name: debian | adding docker repo
  apt_repository:
    repo: "{{ docker_ubuntu_repo_info['repo'] }}"
    state: present
  become: true

# We remove docker-engine as this is old package to install. The new package is
# docker-ce
- name: debian | uninstalling old docker package (if exists)
  apt:
    name: "{{ item }}"
    state: "absent"
    purge: yes
  become: true
  with_items:
    - 'docker-engine'
    - 'lxc-docker'

- name: debian | installing docker pre-reqs
  apt:
    name: "linux-image-extra-{{ ansible_kernel }}"
    state: present
  become: true
  when: >
        ansible_distribution == "Ubuntu" and
        (ansible_distribution_version >= '14.04')

- name: debian | installing docker
  apt:
    name: "docker-ce={{ docker_version_debian }}"
    state: "present"
  become: true

- name: debian | setting grub memory limit (if set)
  lineinfile:
    dest: /etc/default/grub
    regexp: "^GRUB_CMDLINE_LINUX_DEFAULT"
    line: 'GRUB_CMDLINE_LINUX_DEFAULT="cgroup_enable=memory swapaccount=1"'
  register: grub_updated
  become: true
  when: >
        docker_set_grub_memory_limit is defined and
        docker_set_grub_memory_limit

- name: debian | updating grub (if updated)
  command: update-grub
  become: true
  when: grub_updated['changed']

- name: debian | installing additonal packages
  apt:
    name: "{{ item }}"
    state: "present"
  become: true
  with_items:
    - bridge-utils
```

{% endraw %}
First thing I found was that for some reason when installing python-pip
using APT it caused issues with Debian Jessie. It would throw and error
when executing PIP commands. This does not happen on Ubuntu Trusty. So
to get around this I had to add the following bit of logic to the
Ansible task(s). Notice the `when: ansible_distribution == ""`. We use
this Ansible fact that is gathered to determine if we are running Debian
or Ubuntu, do not confuse this with `ansible_os_family == "Debian"` as
both Debian and Ubuntu will report this.

Ubuntu:

```yaml
when: ansible_distribution == "Ubuntu"
```

Debian:

```yaml
when: ansible_distribution == "Debian"
```

Below is the code that is part of the task mentioned above.
{% raw %}

```yaml
- name: uninstall python-pip (If installed) - Debian
  apt:
    name: python-pip
    state: absent
  when: ansible_distribution == "Debian"

- name: debian | installing python packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - python-pip
  when: ansible_distribution == "Ubuntu"

- name: debian | installing python packages
  easy_install:
    name: "{{ item }}"
#    state: present
  with_items:
    - pip
  when: ansible_distribution == "Debian"
```

{% endraw %}
Now we could change both to use easy_install: as our module to use from
an Ansible perspective but...

Another little cool fact that can be used when adding the Docker APT
repositories for Debian and Ubuntu is to use the following Ansible facts
gathered:

```yaml
ansible_distribution
ansible_distribution_release
```

And we can use these as the following defined var in: (adding **\|
lower** will ensure that the ansible_distribution will be ubuntu/debian
instead of Ubuntu/Debian so that it does not cause issues with adding
the APT repositories)

**_defaults/main.yml_**
{% raw %}

```yaml
docker_ubuntu_repo_info:  #defines docker ubuntu repo info for installing from
  - id: 58118E89F3A912897C070ADBF76221572C52609D
    keyserver: hkp://p80.pool.sks-keyservers.net:80
    repo: "deb https://apt.dockerproject.org/repo {{ ansible_distribution | lower }}-{{ ansible_distribution_release }} main"
```

{% endraw %}
**tasks/debian.yml**
{% raw %}

```yaml
- name: debian | adding docker repo
  apt_repository:
    repo: "{{ item.repo }}"
    state: present
  with_items: docker_ubuntu_repo_info
```

{% endraw %}
Using these variables will yield the following based on OS.

`Ubuntu`:
{% raw %}

```yaml
{{ ansible_distribution }} will be Ubuntu
{{ ansible_distribution_release }} will be trusty
```

{% endraw %}
`Debian`:
{% raw %}

```yaml
{{ ansible_distribution }} will be Debian
{{ ansible_distribution_release }} will be jessie
```

{% endraw %}

## UFW

When I began to configure the UFW (Uncomplicated Firewall) for some
firewall rules to be used with Docker I ran into an issue on Debian
Jessie stating that an invalid syntax error occurred when defining the
routed default policy. Again, this works just fine on Ubuntu but not so
much on Debian Jessie. After attempting a few different approaches to
using the ufw Ansible module I finally pulled up the man page on UFW for
Debian Jessie and Ubuntu. Well there is the issue right there. For some
reason on Debian Jessie the UFW settings do not allow defining the
default routed policy whereas on Ubuntu it does. Go figure.
[ansible-ufw](https://github.com/mrlesmithjr/ansible-ufw)

So how do we get around this one? It ended up being fairly simple but
then again...Why do I need to do this?

Below is the bit of code that allows us to get around this (for now).
Again we will use the ansible_distribution fact to determine if we are
running Ubuntu or Debian when our policy direction is set to routed.\
**defaults/main.yml**

```yaml
ufw_policies:  #defines default policy for incoming, outgoing and routed (forwarded) traffic...allow, deny or reject
  - direction: incoming
    policy: deny
  - direction: outgoing
    policy: allow
  - direction: routed
    policy: deny
```

**tasks/configure_firewall.yml**
{% raw %}

```yaml
- name: configure_firewall | setting default UFW policies (incoming, outgoing)
  ufw:
    direction: "{{ item.direction }}"
    policy: "{{ item.policy }}"
  with_items: ufw_policies
  when: item.direction != "routed"

- name: configure_firewall | setting default UFW policies (routed)
  ufw:
    direction: "{{ item.direction }}"
    policy: "{{ item.policy }}"
  with_items: ufw_policies
  when: item.direction == "routed" and ansible_distribution != "Debian"
```

{% endraw %}
And that is all for now. I will be returning to this post and updating
from time to time as I find additional gotchas.

Enjoy!
