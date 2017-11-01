---
  title: Ansible - Provision Docker Swarm Mode (1.12)
---

Lately I have been working quite a bit with the latest Docker Swarm Mode
released in Docker 1.12 and so far it has been pretty awesome. The days
of spinning up a Docker Swarm cluster with all of the
complexity (Consul, Registrator, and etc.) are old-school now. To do a
comparison you can checkout a great post by Scott Lowe
[here](http://blog.scottlowe.org/2015/03/06/running-own-docker-swarm-cluster/) and
also checkout a [Vagrant](https://www.vagrantup.com/) lab that I put
together for learning almost a year ago
[here](https://github.com/mrlesmithjr/vagrant-ansible-docker). With the
latest release of Docker 1.12 they have made the process SO much easier
and much less complex. To read up on the functionality and ease head
over [here](https://docs.docker.com/engine/swarm/). Now the purpose of
this post is not to go step by step on how easy it now is to provision a
cluster but rather to take it to another level. Let's automate the
provisioning using [Ansible](https://www.ansible.com/) instead. This
allows for a much more consistent and predictable provisioning method as
well as the ability to easily scale your cluster(s). In addition to this
post I highly recommend you fork or clone my [GitHub
repo](https://github.com/mrlesmithjr/vagrant-ansible-docker-swarm) to
spin up a 7-node Docker Swarm Mode cluster completely automated and
provisioned using [Ansible](https://www.ansible.com/) and
[Vagrant](https://www.vagrantup.com/). You can read more about the usage
and some examples within that repo as well. So what I WILL cover in this
post is the process along with examples of how to use Ansible to
provision a Docker Swarm Mode cluster.

-   Assumptions:
    -   You have already provisioned your nodes in which will comprise
        your Docker Swarm Mode cluster
        -   OS Installed (_Ubuntu 16.04_ in my case) as well as Docker
            Engine installed...Checkout my
            [Ansible](https://www.ansible.com/) role for this as well
            [here](https://github.com/mrlesmithjr/ansible-docker)
    -   Identified the interface which will be used for all Docker Swarm
        Mode communications and such (_enp0s8_ in my case)

So the first thing we will ensure is correct is our Ansible inventory
and groups. We need to define the overall Docker nodes group
(_docker-nodes_) as well as which nodes are considered managers
(_docker-swarm-managers_) and which nodes are workers
(_docker-swarm-workers_). Below is an example of the Ansible inventory
that I use:

```bash
[docker-nodes]
node[0:6]

[docker-swarm-managers]
node[0:2]

[docker-swarm-workers]
node[3:6]
```

Now we need to define some Ansible variables which are required to
successfully provision our cluster:

{% raw %}

```yaml
docker_swarm_addr: "{{ hostvars[inventory_hostname]['ansible_' + docker_swarm_interface]['ipv4']['address'] }}"
docker_swarm_cert_expiry: '2160h0m0s' # Validity period for node certificates (default 2160h0m0s)
docker_swarm_dispatcher_heartbeat_duration: '5s' # Dispatcher heartbeat period (default 5s)
docker_swarm_interface: "enp0s8"
docker_swarm_managers_ansible_group: 'docker-swarm-managers'
docker_swarm_networks:
  - name: 'my_net'
    driver: 'overlay'
    state: 'present'
  - name: 'test'
    driver: 'overlay'
    state: 'absent'
docker_swarm_primary_manager: '{{ groups[docker_swarm_managers_ansible_group][0] }}'
# docker_swarm_primary_manager: 'node0'
docker_swarm_task_history_limit: '5' # Task history retention limit (default 5)
docker_swarm_workers_ansible_group: 'docker-swarm-workers'
docker_swarm_port: "2377"
```

{% endraw %}

As you can see from the above we can define which node will be
considered as the _docker_swarm_primary_manager_ in two different
ways. The first (default) method is to use the Ansible inventory group
name and choose the first node in that group or the second method would
be to just define the actual node name. Some of the other variables are
to define/update actual Docker Swarm cluster settings (default values by
default) as well as defining some Docker networks to manage.

And now for the actual Ansible playbook that we will be using to make
all of the magic happen behind this:
{% raw %}

```yaml
---
- hosts: docker-nodes
  become: true
  vars:
    docker_swarm_addr: "{{ hostvars[inventory_hostname]['ansible_' + docker_swarm_interface]['ipv4']['address'] }}"
    docker_swarm_cert_expiry: '2160h0m0s' # Validity period for node certificates (default 2160h0m0s)
    docker_swarm_dispatcher_heartbeat_duration: '5s' # Dispatcher heartbeat period (default 5s)
    docker_swarm_interface: "enp0s8"
    # docker_swarm_managers_ansible_group: 'docker-swarm-managers'
    docker_swarm_networks:
      - name: 'my_net'
        driver: 'overlay'
        state: 'present'
      - name: 'test'
        driver: 'overlay'
        state: 'absent'
    docker_swarm_primary_manager: '{{ groups[docker_swarm_managers_ansible_group][0] }}'
    # docker_swarm_primary_manager: 'node0'
    docker_swarm_task_history_limit: '5' # Task history retention limit (default 5)
    # docker_swarm_workers_ansible_group: 'docker-swarm-workers'
    docker_swarm_port: "2377"
  tasks:
    - name: docker_swarm | Installing EPEL Repo (RedHat)
      yum:
        name: "epel-release"
        state: "present"
      when: >
            ansible_os_family == "RedHat" and
            ansible_distribution != "Fedora"

    - name: docker_swarm | Installing Pre-Reqs
      apt:
        name: "python-pip"
        state: "present"
      when: ansible_os_family == "Debian"

## Installing these for future functionality
    - name: docker_swarm | Installing Python Pre-Reqs
      pip:
        name: "{{ item }}"
        state: "present"
      with_items:
        - 'docker-py'
##

    - name: docker_swarm | Ensuring Docker Engine Is Running
      service:
        name: "docker"
        state: "started"

    - name: docker_swarm | Checking Swarm Mode Status
      command: "docker info"
      register: "docker_info"
      changed_when: false

    - name: docker_swarm | Init Docker Swarm Mode On First Manager
      command: >
              docker swarm init
              --listen-addr {{ docker_swarm_addr }}:{{ docker_swarm_port }}
              --advertise-addr {{ docker_swarm_addr }}
      when: >
            'Swarm: inactive' in docker_info.stdout and
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Capturing Docker Swarm Worker join-token
      command: "docker swarm join-token -q worker"
      changed_when: false
      register: "docker_swarm_worker_token"
      when: >
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Capturing Docker Swarm Manager join-token
      command: "docker swarm join-token -q manager"
      changed_when: false
      register: "docker_swarm_manager_token"
      when: >
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Defining Docker Swarm Manager Address
      set_fact:
        docker_swarm_manager_address: "{{ docker_swarm_addr }}:{{ docker_swarm_port }}"
      changed_when: false
      when: >
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Defining Docker Swarm Manager Address
      set_fact:
        docker_swarm_manager_address: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_manager_address'] }}"
      changed_when: false
      when: >
            inventory_hostname != docker_swarm_primary_manager

    - name: docker_swarm | Defining Docker Swarm Manager join-token
      set_fact:
        docker_swarm_manager_token: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_manager_token'] }}"
      changed_when: false
      when: >
            inventory_hostname != docker_swarm_primary_manager

    - name: docker_swarm | Defining Docker Swarm Worker join-token
      set_fact:
        docker_swarm_worker_token: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_worker_token'] }}"
      changed_when: false
      when: >
            inventory_hostname != docker_swarm_primary_manager

    - name: docker_swarm | Joining Additional Docker Swarm Managers To Cluster
      command: >
              docker swarm join
              --listen-addr {{ docker_swarm_addr }}:{{ docker_swarm_port }}
              --advertise-addr {{ docker_swarm_addr }}
              --token {{ docker_swarm_manager_token.stdout }}
              {{ docker_swarm_manager_address }}
      when: >
            inventory_hostname != docker_swarm_primary_manager and
            inventory_hostname not in groups[docker_swarm_workers_ansible_group] and
            'Swarm: active' not in docker_info.stdout and
            'Swarm: pending' not in docker_info.stdout

    - name: docker_swarm | Joining Docker Swarm Workers To Cluster
      command: >
             docker swarm join
             --listen-addr {{ docker_swarm_addr }}:{{ docker_swarm_port }}
             --advertise-addr {{ docker_swarm_addr }}
             --token {{ docker_swarm_worker_token.stdout }}
             {{ docker_swarm_manager_address }}
      when: >
            inventory_hostname in groups[docker_swarm_workers_ansible_group] and
            'Swarm: active' not in docker_info.stdout and
            'Swarm: pending' not in docker_info.stdout

    - name: docker_swarm | Capturing Docker Swarm Networks
      command: "docker network ls"
      changed_when: false
      register: "docker_networks"
      when: >
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Creating Docker Swarm Networks
      command: "docker network create --driver {{ item.driver }} {{ item.name }}"
      with_items: '{{ docker_swarm_networks }}'
      when: >
            inventory_hostname == docker_swarm_primary_manager and
            item.state|lower == "present" and
            item.name not in docker_networks.stdout

    - name: docker_swarm | Removing Docker Swarm Networks
      command: "docker network rm {{ item.name }}"
      with_items: '{{ docker_swarm_networks }}'
      when: >
            inventory_hostname == docker_swarm_primary_manager and
            item.state|lower == "absent" and
            item.name in docker_networks.stdout

## Below is for Ansible 2.2
    # - name: docker_swarm | Managing Docker Swarm Networks
    #   docker_network:
    #     name: "{{ item.name }}"
    #     driver: "{{ item.driver }}"
    #     state: "{{ item.state }}"
    #   with_items: '{{ docker_swarm_networks }}'
    #   when: >
    #         inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Updating Docker Swarm Dispatch Heartbeat Duration
      command: "docker swarm update --dispatcher-heartbeat {{ docker_swarm_dispatcher_heartbeat_duration }}"
      when: >
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Updating Docker Swarm Certificate Expiry Duration
      command: "docker swarm update --cert-expiry {{ docker_swarm_cert_expiry }}"
      when: >
            inventory_hostname == docker_swarm_primary_manager

    - name: docker_swarm | Updating Docker Swarm Task History Limit
      command: "docker swarm update --task-history-limit {{ docker_swarm_task_history_limit }}"
      when: >
            inventory_hostname == docker_swarm_primary_manager
```

{% endraw %}
As you can see we are actually capturing both the manager and worker
tokens for the cluster as well. This is the key for success here. So how
are we doing this? Let's take a look at the tasks that accomplish this.

First we need to capture the tokens on our
docker_swarm_primary_manager:

{% raw %}

```yaml
- name: docker_swarm | Capturing Docker Swarm Worker join-token
  command: "docker swarm join-token -q worker"
  changed_when: false
  register: "docker_swarm_worker_token"
  when: >
        inventory_hostname == docker_swarm_primary_manager

- name: docker_swarm | Capturing Docker Swarm Manager join-token
  command: "docker swarm join-token -q manager"
  changed_when: false
  register: "docker_swarm_manager_token"
  when: >
        inventory_hostname == docker_swarm_primary_manager
```

{% endraw %}

If you notice from above we are literally just running the docker swarm
command to query our join-token for the manager and worker tokens and
registering those facts. Easy as that!

Next we need to define the facts for both join-tokens in order for our
additional nodes to successfully join our Swarm cluster. And we do that
with the following tasks:

{% raw %}

```yaml
- name: docker_swarm | Defining Docker Swarm Manager join-token
  set_fact:
    docker_swarm_manager_token: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_manager_token'] }}"
  changed_when: false
  when: >
        inventory_hostname != docker_swarm_primary_manager

- name: docker_swarm | Defining Docker Swarm Worker join-token
  set_fact:
    docker_swarm_worker_token: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_worker_token'] }}"
  changed_when: false
  when: >
        inventory_hostname != docker_swarm_primary_manager
```

{% endraw %}

Now that these facts are registered we can now leverage these on our
additional Swarm nodes.

Some additional tasks we are able to do are to also manage our Docker
networks including overlay networks for our Swarm cluster. We can also
update some additional settings for our Docker Swarm cluster which we do
in the last three tasks of the playbook.

So there you have it. An easy way to provision a Docker Swarm (1.12)
mode cluster using Ansible allowing for easily scaled out provisioning.

Stay tuned for additional posts coming in the future in regards to some
additional Docker Swarm mode goodness.

Enjoy!
