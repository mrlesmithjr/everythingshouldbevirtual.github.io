---
  title: Ansible - RabbitMQ Cluster
---

> UPDATE: As of 08/06/2015 the following role can be found
> [here](https://galaxy.ansible.com/list#/roles/4594) on Ansible Galaxy.
> The below information was updated based on this new role available at
> the above link and verified to work..However, this information below
> may be out of date so please grab the latest role from the Ansible
> Galaxy link above.

In this post I will be setting up a clustered RabbitMQ environment for
logstash message queuing. Again I will be keeping this short and sweet.
This might come in use for someone else as well. This setup will join
each node to the cluster, setup a logstash exchange and logstash queue
and then also configure the binding between our logstash exchange and
logstash queue. It will also configure the ha-policy to spread the queue
across all nodes and sync them automatically.

We will be starting with the following `group_vars/app` to setup our
variables..Make sure to adjust to fit the needs of your environment...

```yaml
config_rabbitmq_ha: true
enable_rabbitmq_clustering: true
erlang_cookie: LSKNKBELKPSTDBBCHETL
rabbitmq_config:
  - queue_name: logstash
    durable: true
    exchange_name: logstash
    type: fanout
    routing_key: logstash
    tags: 'ha-mode=all,ha-sync-mode=automatic'
rabbitmq_master: ans-test-1
```

Within our roles tree for rabbitmq it looks like below ![Screen Shot 2015-04-13 at 9.32.12 AM](../../assets/Screen-Shot-2015-04-13-at-9.32.12-AM-300x164.png)

Below will be the contents of each file within the tree from the
screenshot above.

`handlers/main.yml`:

```yaml
---
- name: restart rabbitmq-server
  service: name=rabbitmq-server state=restarted
```

`tasks/main.yml`:

```yaml
---
- include: debian.yml
  when: ansible_os_family == "Debian"

- name: checking to see if already clustered
  stat: path=/etc/rabbitmq/clustered
  register: clustered

- include: rabbitmq_clustering.yml
  when: enable_rabbitmq_clustering and (clustered.stat.exists != True)

- include: rabbitmq_ha_config.yml
  when: config_rabbitmq_ha and enable_rabbitmq_clustering
```

`tasks/debian.yml`:

```yaml
---
- name: debian | adding RabbitMQ public GPG key to the apt repo
  apt_key: url=http://www.rabbitmq.com/rabbitmq-signing-key-public.asc state=present

- name: debian | adding RabbitMQ repo
  apt_repository: repo='deb http://www.rabbitmq.com/debian/ testing main' update_cache=no state=present

- name: debian | installing RabbitMQ server
  apt: name={{ item }} state=present
  with_items:
    - rabbitmq-server

- name: debian | enabling the RabbitMQ Management Console
  rabbitmq_plugin: names=rabbitmq_management state=enabled
  notify: restart rabbitmq-server

- name: debian | ensuring that the RabbitMQ service is running
  service: name=rabbitmq-server state=started enabled=yes
```

`tasks/rabbitmq_clustering.yml`:
{% raw %}

```yaml
---
- name: rabbitmq_clustering | stopping rabbitmq app
  command: rabbitmqctl stop_app

- name: rabbitmq_clustering | resetting rabbitmq app
  command: rabbitmqctl reset

- name: rabbitmq_clustering | stopping rabbitmq-server
  service: name=rabbitmq-server state=stopped

- name: rabbitmq_clustering | copy erlang cookie
  template: src=erlang.cookie.j2 dest=/var/lib/rabbitmq/.erlang.cookie owner=rabbitmq group=rabbitmq mode=0400 backup=yes

- name: rabbitmq_clustering | restarting rabbitmq-server on master
  service: name=rabbitmq-server state=restarted
  when: inventory_hostname == "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | starting rabbitmq app on master
  command: rabbitmqctl start_app
  register: cluster_master
  when: inventory_hostname == "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | sending sigterm to any running rabbitmq processes
  shell: pkill -u rabbitmq || true
  when: inventory_hostname != "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | restarting rabbitmq-server
  service: name=rabbitmq-server state=restarted
  when: inventory_hostname != "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | stopping rabbitmq app
  command: rabbitmqctl stop_app
  when: inventory_hostname != "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | resetting rabbitmq app
  command: rabbitmqctl reset
  when: inventory_hostname != "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | joining rabbitmq cluster
  command: rabbitmqctl join_cluster 'rabbit@{{ rabbitmq_master }}'
  register: cluster_joined
  when: inventory_hostname != "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | starting rabbitmq app
  command: rabbitmqctl start_app
  when: inventory_hostname != "{{ rabbitmq_master }}"

- name: rabbitmq_clustering | marking as clustered
  file: path=/etc/rabbitmq/clustered state=touch
  when: cluster_master.changed or cluster_joined.changed
```

{% endraw %}

`tasks/rabbitmq_ha_config.yml`:
{% raw %}

```yaml
---
- name: rabbitmq_ha_config | install rabbitMQ admin
  shell: wget http://guest:guest@localhost:55672/cli/rabbitmqadmin

- name: rabbitmq_ha_config | moving the rabbitMQ Admin
  shell: mv rabbitmqadmin /usr/sbin

- name: rabbitmq_ha_config | making executable rabbitMQ Admin
  shell: chmod +x /usr/sbin/rabbitmqadmin
  notify: restart rabbitmq-server

- name: rabbitmq_ha_config | creating queue(s)
  command: rabbitmqadmin declare queue name={{ item.queue_name }} durable={{ item.durable|lower }}
  run_once: true
  with_items: rabbitmq_config

- name: rabbitmq_ha_config | setting up ha on queue(s)
  rabbitmq_policy: name='ha-all' pattern='{{ item.queue_name }}' tags="{{ item.tags }}" state=present
  run_once: true
  with_items: rabbitmq_config

- name: rabbitmq_ha_config | creating exchange(s)
  command: rabbitmqadmin declare exchange name={{ item.exchange_name }} type={{ item.type }}
  run_once: true
  with_items: rabbitmq_config

- name: rabbitmq_ha_config | creating binding(s)
  command: rabbitmqadmin declare binding source={{ item.exchange_name }} destination_type="queue" destination={{ item.queue_name }} routing_key={{ item.routing_key }}
  run_once: true
  with_items: rabbitmq_config
```

{% endraw %}

`templates/erlang.cookie.j2`:
{% raw %}

```yaml
{{ erlang_cookie }}
```

{% endraw %}

When you are all done this is what you should see in your RabbitMQ web management UI.

![Screen Shot 2015-04-13 at 9.41.31 AM](../../assets/Screen-Shot-2015-04-13-at-9.41.31-AM-280x300.png)

And there you have it. You now have a highly available RabbitMQ cluster
all setup by Ansible.

Enjoy!
