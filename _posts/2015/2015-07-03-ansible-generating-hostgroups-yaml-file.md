---
  title: Ansible - Generating Host/Groups YAML file
---

As I have been working on a nice little project of mine (More on that in
the near future) I came across the need to take my hosts inventory INI
file and get it into a usable yaml file. I had done this in the past but
it was rather ugly using sed, awk and etc. I finally came up with a nice
solution and figured that I would share it with others who may have a
need for something similar.

> My **_hosts_** inventory was like below:

```bash
jumpbox

[ansible-servers]
ansible

[ansible-test-servers]
ans-test-[1:5]

[cacti-servers]
cactimon

[ci-pipeline]
ci-pipeline

[cmdb-servers]
cmdb

[desktops]
home-ms-7693
home-Latitude-E6530

[elk-p-nodes]
elk-p-haproxy-[1:2]
elk-p-broker-[1:3]
elk-p-es-[1:3]
elk-p-pre-processor-[1:2]
elk-p-processor-[1:5]

[elk-p-haproxy-nodes]
elk-p-haproxy-[1:2]

[elk-p-broker-nodes]
elk-p-broker-[1:3]

[elk-p-es-nodes]
elk-p-es-[1:3]

[elk-p-pre-processor-nodes]
elk-p-pre-processor-[1:2]

[elk-p-processor-nodes]
elk-p-processor-[1:5]

[gitlab-servers]
gitlab

[grafana]
grafana

[graylog-servers]
graylog

[ip-services]
ns[1:2]
ns-1-2-clust-mgmr

[nagios]

[nameservers]
ns[1:2]

[plex-servers]
plexmediaserver

[sensu-servers]
sensu

[smtp]

[smtp-servers]
smtp

[squid-servers]
squid-[1:2]

[tftp-server]
tftpbuild-serv

[vmcreate-Ubuntu]
ans-test-[1:5]
elk-p-processor-5
cmdb

[vmcreate-Windows]

[vmdelete]
```

> Create a playbook called **_create_host_groups_inventory.yml_**
> with the below:
> {% raw %}

```yaml
- hosts: all
  sudo: true
  remote_user: remote
  vars:
    - db_inventory_dir: ./db_inventory
    - db_inventory_file: ./db_inventory.yml
  tasks:
    - name: creating db_inventory_dir
      file:
        path: "{{ db_inventory_dir }}"
        state: directory
      delegate_to: localhost
      run_once: true

    - name: removing existing inventory files
      file:
        path: "{{ item }}"
        state: absent
      delegate_to: localhost
      with_items:
        - "{{ db_inventory_dir }}/{{ ansible_hostname }}.yml"
        - "{{ db_inventory_file }}"

    - name: creating yaml list of groups/hosts
      template:
        src: "db_inventory_list.yml.j2"
        dest: "{{ db_inventory_dir }}/{{ ansible_hostname }}.yml"
      delegate_to: localhost

    - name: merging all host inventory groups into one
      assemble:
        src: "{{ db_inventory_dir }}/"
        dest: "{{ db_inventory_file }}"
      delegate_to: localhost
      run_once: true

    - name: adding list var to inventory file
      lineinfile:
        dest: "{{ db_inventory_file }}"
        regexp: "^host_groups"
        line: "host_groups{{ ':' }}"
        insertbefore: BOF
      delegate_to: localhost
      run_once: true

    - name: adding yaml formatting to inventory file
      lineinfile:
        dest: "{{ db_inventory_file }}"
        regexp: "^---" line="---"
        insertbefore: BOF
      delegate_to: localhost
      run_once: true

    - name: chmodding inventory file
      file:
        path: "{{ db_inventory_file }}"
        mode: 0775
      delegate_to: localhost
      run_once: true
```

{% endraw %}
Create a template called `db_inventory_list.yml.j2` with below:
(\*\*Spacing is key when copying this file)
{% raw %}

```yaml
  - name: {{ ansible_hostname }}
    groups:
{% for group in group_names %}
      - {{ group }}
{% endfor %}
```

{% endraw %}
Now execute the following:

```bash
ansible-playbook -i hosts create_host_groups_inventory.yml
```

Once it is complete cat the db_inventory.yml and you will see the
following.

```yaml
---
host_groups:
  - name: ans-test-1
    groups:
      - ansible-test-servers
      - vmcreate-Ubuntu

  - name: ans-test-2
    groups:
      - ansible-test-servers
      - vmcreate-Ubuntu

  - name: ans-test-3
    groups:
      - ansible-test-servers
      - vmcreate-Ubuntu

  - name: ans-test-4
    groups:
      - ansible-test-servers
      - vmcreate-Ubuntu

  - name: ans-test-5
    groups:
      - ansible-test-servers
      - vmcreate-Ubuntu

  - name: ansible
    groups:
      - ansible-servers

  - name: ci-pipeline
    groups:
      - ci-pipeline

  - name: cmdb
    groups:
      - cmdb-servers
      - vmcreate-Ubuntu

  - name: elk-p-broker-1
    groups:
      - elk-p-broker-nodes
      - elk-p-nodes

  - name: elk-p-broker-2
    groups:
      - elk-p-broker-nodes
      - elk-p-nodes

  - name: elk-p-broker-3
    groups:
      - elk-p-broker-nodes
      - elk-p-nodes

  - name: elk-p-es-1
    groups:
      - elk-p-es-nodes
      - elk-p-nodes

  - name: elk-p-es-2
    groups:
      - elk-p-es-nodes
      - elk-p-nodes

  - name: elk-p-es-3
    groups:
      - elk-p-es-nodes
      - elk-p-nodes

  - name: elk-p-haproxy-1
    groups:
      - elk-p-haproxy-nodes
      - elk-p-nodes

  - name: elk-p-haproxy-2
    groups:
      - elk-p-haproxy-nodes
      - elk-p-nodes

  - name: elk-p-pre-processor-1
    groups:
      - elk-p-nodes
      - elk-p-pre-processor-nodes

  - name: elk-p-pre-processor-2
    groups:
      - elk-p-nodes
      - elk-p-pre-processor-nodes

  - name: elk-p-processor-1
    groups:
      - elk-p-nodes
      - elk-p-processor-nodes

  - name: elk-p-processor-2
    groups:
      - elk-p-nodes
      - elk-p-processor-nodes

  - name: elk-p-processor-3
    groups:
      - elk-p-nodes
      - elk-p-processor-nodes

  - name: elk-p-processor-4
    groups:
      - elk-p-nodes
      - elk-p-processor-nodes

  - name: elk-p-processor-5
    groups:
      - elk-p-nodes
      - elk-p-processor-nodes
      - vmcreate-Ubuntu

  - name: gitlab
    groups:
      - gitlab-servers

  - name: grafana
    groups:
      - grafana

  - name: home-ms-7693
    groups:
      - desktops

  - name: jumpbox
    groups:
      - ungrouped

  - name: ns-1-2-clust-mgmr
    groups:
      - ip-services

  - name: ns1
    groups:
      - ip-services
      - nameservers

  - name: ns2
    groups:
      - ip-services
      - nameservers

  - name: plexmediaserver
    groups:
      - plex-servers

  - name: sensu
    groups:
      - sensu-servers

  - name: smtp
    groups:
      - smtp-servers

  - name: squid-1
    groups:
      - squid-servers

  - name: squid-2
    groups:
      - squid-servers

  - name: tftpbuild-serv
    groups:
      - tftp-server
```

There you have it. A nicely formatted yaml inventory list to consume as
your needs require.

Enjoy!
