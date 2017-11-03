---
  title: Ansible - Highly Available ELK Stack
---

A little over a year ago I provided installation scripts along with
[this](https://everythingshouldbevirtual.com/highly-available-elk-elasticsearch-logstash-kibana-setup)
post to help setup a completely redundant ELK Stack. This post has
definitely been one of my most popular posts that I have ever put out
and lot's of great feedback provided. Over the past few months I have
been working on getting all of this functionality and then some ported
over to [Ansible](http://ansible.com) which has proven to be amazing to
say the least. However the missing piece has been providing this out to
the community for consumption as well as getting additional feedback in
areas that could be improved on. So I am now putting this all together
(Hopefully in a consumable way) and providing this to the great
community. As always this will be a working guide on setting this all up
as I am sure that I will miss pieces here and there as I release these
playbooks. I have spent a good bit of time trying to make these
playbooks as customizable as I can allowing for them to fit within any
environment. I am of course open to any ideas or thoughts on how to make
these even better.

All of this will be contained in my
[GitHub](https://github.com/mrlesmithjr/Ansible) repository. A note on
this is that I have created one repository for all Ansible roles in
order to use them across different solutions as well as have a
repository that would be all inclusive for a complete site setup which
would include additional environments other than just ELK. As I share
out additional roles I will be building on the foundation of a complete
site, meaning that variables that should carry out over all roles will
be contained in **_group_vars/all_** directory**_. _**I prefer to use
individual files for specific variables instead of placing all of them
in a single group_vars/all file. But of course you may choose to do
differently.

**UPDATE - 04-17-2016**

As you work through this setup it has come to my attention that this
post is extremely out of date. So YMMV as you attempt to install ELK. I
intend on publishing a more current post on setting up ELK which in most
cases is even easier than the process here including the Ansible roles.
If you would like to clone and leverage much more current versions of
each ELK role or would like to just have them as a reference in case you
run into issues following this post. You can visit each of the newer
Ansible roles from below. Hope this helps!

The main Ansible role with some details included can be found
[here](https://github.com/mrlesmithjr/ansible-elkstack).

<https://github.com/mrlesmithjr/ansible-elk-kibana>
<https://github.com/mrlesmithjr/ansible-elk-processor>
<https://github.com/mrlesmithjr/ansible-elk-haproxy>
<https://github.com/mrlesmithjr/ansible-elk-pre-processor>
<https://github.com/mrlesmithjr/ansible-elk-es>
<https://github.com/mrlesmithjr/ansible-elk-broker>

Example of `group_vars/all`:

```bash
administrator@ansible:~/ansible/group_vars/all$ ls -l
total 44
-rw-rw-r-- 1 administrator administrator 5753 Jul 20 17:31 accounts
-rw-rw-r-- 1 administrator administrator 1073 Jul 20 17:31 email
-rw-rw-r-- 1 administrator administrator  284 Jul 20 17:31 gitlab
-rw-rw-r-- 1 administrator administrator  468 Jul 20 17:31 monitoring
-rw-rw-r-- 1 administrator administrator 1160 Jul 20 17:31 network
-rw-rw-r-- 1 administrator administrator  214 Jul 20 17:31 rundeck
-rw-rw-r-- 1 administrator administrator  139 Jul 20 17:31 security
-rw-rw-r-- 1 administrator administrator 2127 Jul 20 17:31 servers
-rw-rw-r-- 1 administrator administrator  233 Jul 20 17:31 time
-rw-rw-r-- 1 administrator administrator  457 Jul 20 17:31 vcenter
```

As I provision nodes in my environment I use a bootstrap playbook and
then a site playbook which will perform many tasks that should be
consistent across all nodes. Feel free to tweak as you see fit but make
sure to modify variables in specific files
in **_group_vars/all_** prior to going any further ensuring that these
variable align with your environment. I keep all specific user account
and passwords in **_group_vars/all/accounts_** therefore when you pull
this file from Github it will not contain much.

`bootstrap.yml`:
{% raw %}

```yaml
---
- hosts: all
  sudo: yes
  roles:
    - { role: change-hostname, when: update_hostname }

- hosts: all
  gather_facts: yes
  sudo: yes
  roles:
    - bootstrap
```

{% endraw %}
`site.yml`:
{% raw %}

```yaml
---
- hosts: all
  sudo: yes
  remote_user: remote
  roles:
    - { role: disable-firewall, when: not enable_firewall }
    - { role: enable-firewall, when: enable_firewall }
    - base
    - ntp
    - rsyslog
    - { role: change-hostname, when: update_hostname }
    - { role: collectd, when: enable_collectd_monitoring }
    - { role: postfix, when: configure_postfix }
    - { role: sensu, when: enable_sensu_monitoring }
    - { role: snmpd, when: enable_snmpd }
    - { role: sysdig, when: install_sysdig is defined and install_sysdig }
    - { role: timezone, when: change_timezone }
    - { role: zabbix-agent, when: enable_zabbix_monitoring }
```

{% endraw %}
So you should now clone my Github repo by doing the following into a
folder of your choosing.

```bash
git clone https://github.com/mrlesmithjr/Ansible.git
```

So with all of the above out of the way let's go ahead and get started.

> Assumption: You have an understanding of using Ansible

## ELK Nodes

In order to make this highly available and scalable you will need to
deploy a number of nodes in groups based on their role. And each role
will have a specific resource requirement. This will be based on a
starting point allowing for you to scale either vertical or horizontal
(preferably horizontal).

The naming that I will be using will have a **_-p_** in the node names
which references production. This allows for you to build out production
and say dev or test using the same playbooks and roles with just some
minor changes to some group_vars/groups. All nodes are based on Ubuntu
14.04 LTS x64.

-   elk-p-broker-nodes
    -   3 nodes
    -   Elasticsearch Master nodes, Redis cache, Kibana WebUI
    -   Elasticsearch min. 2 master nodes up in cluster required
    -   1vCPU, 4GB RAM, 36GB hard drive
-   elk-p-es-nodes
    -   3 nodes
    -   Elasticsearch Data nodes
    -   2vCPU,4GB RAM, 250GB hard drive
-   elk-p-pre-processor-nodes
    -   2 nodes
    -   Logstash inputs, pre-processing, multiline filtering
    -   Send output to the broker-nodes via Redis
    -   1vCPU, 1GB RAM, 36GB hard drive
-   elk-p-processor-nodes
    -   4 nodes
    -   Logstash parsers
    -   Gather input from broker nodes via Redis
    -   Send output to the es-nodes via Elasticsearch
    -   2vCPU, 2GB RAM, 36GB hard drive
-   elk-p-haproxy-nodes
    -   2 nodes
    -   KeepAliveD VIP, Load Balancers for all of the ELK Stack
    -   Also gather UDP syslog for devices that cannot be configured for
        TCP
    -   1vCPU, 512MB RAM, 36GB hard drive

Once you have all of these deployed you will create or add to your
Ansible inventory as follows. The group elk-p-nodes will include all of
the nodes part of the setup in order to apply variables that apply to
all nodes allowing us to get more granular with variables in the
specific groups. Or you may use the included hosts file included in the
Github repo.

`hosts`:

```bash
[elk-p-nodes]
elk-p-haproxy-[1:2]
elk-p-broker-[1:3]
elk-p-es-[1:3]
elk-p-pre-processor-[1:2]
elk-p-processor-[1:4]

[elk-p-haproxy-nodes]
elk-p-haproxy-[1:2]

[elk-p-broker-nodes]
elk-p-broker-[1:3]

[elk-p-es-nodes]
elk-p-es-[1:3]

[elk-p-pre-processor-nodes]
elk-p-pre-processor-[1:2]

[elk-p-processor-nodes]
elk-p-processor-[1:4]
```

Now either create or modify the following group_vars/group names to fit
your requirements. (Only groups with specific settings pertaining to ELK
will be shown below and are only shown here for a reference. Each of
these are included in the github repository.)

-   group_vars/elk-p-broker-nodes
-   group_vars/elk-p-es-nodes
-   group_vars/elk-p-haproxy-nodes

```yaml
syslog_servers:
  - name: '{{ logstash_server_fqdn }}'
    port: 514
    proto: tcp
  - name: 'logstash-dev.{{ pri_domain_name }}'
    port: 514
    proto: udp
```

`group_vars/elk-p-nodes`:

```yaml
additional_logstash_workers: true  #true=add additional workers defined in logstash_workers | false=disable
config_logstash: true  #defines if logstash should be configured or not...this should be set to true unless there is a reason not to
config_rabbitmq_ha: '{{ use_rabbitmq }}'  #only define if using rabbitmq instead of redis for broker nodes and rabbitmq ha is required for redundancy of queues **recommended if using rabbitmq
enable_firewall: false
enable_rabbitmq_clustering: '{{ use_rabbitmq }}'  #only define if using rabbitmq instead of redis for broker nodes and your requirements are for clustering of rabbitmq nodes **recommended if using rabbitmq
es_cluster_name: logstash-prod  #define the elasticsearch cluster name to configure
es_config_nfs: false  #configure and NFS mount to setup for archiving of elasticsearch data - has to be configured using the elasticsearch API or another method **recommended for long term archiving
es_curator_close_after_days: 14  #defined the number of days to keep elasticsearch indexes open
es_curator_max_keep_days: 30  #defines data retention policy for elasticsearch...purges data to keep disk space in order... **recommend aligning this with NFS archiving
es_dest: /etc/elasticsearch/elasticsearch.yml
es_fielddata_cache_size: 40%  #elasticsearch tweak...research before tweaking this
es_min_master_nodes: 2  #defines the minimum number of elasticsearch master nodes to keep from split-brain scenario in cluster....adjust based on number of elk-broker-nodes deployed. should be at least 1 more than half of nodes
es_nfs_mount: 10.0.101.51:/volumes/HD-Pool/elasticsearch-snapshots  #define NFS mount to configure if setting up for archiving
es_nfs_mount_opts: defaults  #adjust NFS mountpoint options...research before changing
es_nfs_mountpoint: /mnt/elasticsearch-snapshots  #define NFS mountpoint to mount es_nfs_mount to
es_replicas: 1  #defines the number of data replicas to maintain in elasticsearch cluster....default is 1...research before changing this
es_shards: 5  #defines the number of data shards to maintain in elasticsearch cluster....default is 5...research before changing this
esxinaming:  #define your VMware ESXi naming standards if used...this should be set to host pattern...example - esxi01.everythingshouldbevirtual.local - define as esxi
  - esxi
hadoop_notifications: '{{ infra_email_notifications }}'
hadoopnaming: '' #define your Hadoop naming standards if used...this should be set to host pattern...example - hd01.everythingshouldbevirtual.local - define as hd
#  - hd  #uncomment and remove '' above if setting
keepalived_router_id: 51  #defines the router_id to configure for keepalived...ensure not to define an already in use router_id if keepalived exists on the same subnet
keepalived_vip: 10.0.101.60  #defines the VIP to be assigned to the cluster...this will be the address to access all components of ELK...create a DNS record for this address...ex - logstash.everythingshouldbevirtual.local
keepalived_vip_hostname: '{{ logstash_server_fqdn }}'  #define the DNS record created for keepalived_vip  #defined in group_vars/all/configs
keepalived_vip_int: eth0  #defines the network interface for keepalived to use for VIP and HAProxy (Load Balancer)
kibana_elasticsearch: 'http://{{ keepalived_vip_hostname }}:9200'  #defines the url to configure Kibana UI to communicate with elasticsearch cluster..should be set to keepalived_vip or keepalived_vip_hostname
kibana_host: 0.0.0.0  #defines Kibana host...should remain as 0.0.0.0 unless other requirements are required...research before changing
kibana_port: 5601
logstash_alerts_domain: '{{ smtp_domain_name }}'  #defines the domain to be applied to email alerts...this may be different than pri_domain_name (group_vars/all/configs) ex. everythingshouldbevirtual.local and everythingshouldbevirtual.com
logstash_alerts_email: 'logstash.alerts@{{ logstash_alerts_domain }}'  #define email address to use for email alerts to be sent from logstash
logstash_config_dir: /etc/logstash/conf.d  #defines the location where logstash configurations will be located....default is /etc/logstash/conf.d
logstash_drop_grokparsefailures: false  #set to true if you want to drop all messages resulting in _grokparsefailure
logstash_enable_alerts: true  #defines if alerts should be enabled...example is email alerts..
logstash_workers: 4  #define the number of logstash worker processes to spawn
netscalernaming: '' #define your Citrix Netscaler naming standards if used...this should be set to host pattern...example - nsvpx01.everythingshouldbevirtual.local - define as nsvpx
#  - nsvpx  #uncomment and remove '' above if setting
nsxnaming: '' #define your VMware NSX naming standards if used...this should be set to host pattern...example - nsx-rt01.everythingshouldbevirtual.local - define as nsx-rt
#  - vShield-edge  #uncomment and remove '' above if setting
#  - nsx-rt  #uncomment and remove '' above if setting
pfsensenaming:  #define your PFSense firewall naming standards if used...this should be set to host pattern...example - pfsense01.everythingshouldbevirtual.local - define as pfsense
  - pfsense
redis_max_memory: 2048  #define the max amount of memory to use for redis
redis_max_memory_policy: allkeys-lru  #defines redis max memory policy...research before changing
redis_server_name: '{{ keepalived_vip_hostname }}'  #define the hostname or IP address of your redis server for elk-pre-processors to send logstash output...should be set to keepalived_vip or keepalived_vip_hostname
rundeck_logstash_host: '{{ logstash_server_fqdn }}'
rundeck_logstash_port: 9700
use_rabbitmq: false  #defines if rabbitmq should be used on elk-broker nodes...either use rabbitmq or redis... **recommend redis
use_redis: true  #defines if redis should be used on elk-broker nodes...either use redis or rabbitmq... **recommend redis
```

`elk-p-pre-processor-nodes`

```yaml
syslog_servers:
  - name: localhost
    port: 10514
    proto: tcp
```

`elk-p-processor-nodes`

Now with **_group_vars_** out of the way you will need to modify or
create host_vars/node name for only the \_elk-p-haproxy_ nodes but
the defaults I have included she be all that is required which basically
only defines KeepAliveD setting specific to each node. Examples below.

`elk-p-haproxy-1`

```yaml
---
keepalived_router_pri: 101
keepalived_vrrp_state: MASTER
```

`elk-p-haproxy-2`

```yaml
---
keepalived_router_pri: 100
keepalived_vrrp_state: BACKUP
```

Now we should be ready to run our elkstack playbook and watch the
building begin. Below is what the playbook looks like which is included
in the Github repo. You will need to modify the remote_user variable to
reflect the account that you will be connecting to all of your nodes
with.

`elkstack_prod.yml`
{% raw %}

```yaml
---
- hosts: elk-p-nodes
  remote_user: remote
  sudo: yes
  roles:
    - { role: disable-firewall, when: not enable_firewall }
    - { role: enable-firewall, when: enable_firewall }
    - elk-network-tweaks

- hosts: elk-p-broker-nodes
  remote_user: remote
  sudo: yes
  roles:
    - { role: redis, when: use_redis }
    - { role: rabbitmq, when: use_rabbitmq }
    - nginx
    - elasticsearch
    - elk-broker
    - elk-kibana
    - elk-tuning

- hosts: elk-p-es-nodes
  remote_user: remote
  sudo: yes
  roles:
    - elasticsearch
    - elk-es
    - elk-tuning

- hosts: elk-p-pre-processor-nodes
  remote_user: remote
  sudo: yes
  roles:
    - logstash
    - elk-pre-processor
    - dnsmasq

- hosts: elk-p-processor-nodes
  remote_user: remote
  sudo: yes
  roles:
    - elasticsearch
    - logstash
    - elk-processor
    - dnsmasq

- hosts: elk-p-haproxy-nodes
  remote_user: remote
  sudo: yes
  roles:
    - logstash
    - haproxy
    - elk-haproxy
```

{% endraw %}
So if all nodes have been provisioned and naming matches the Ansible
group_vars, host_vars and the Ansible inventory file you should be
ready to run the following and watch everything get built.

```bash
ansible-playbook -i hosts elkstack_prod.yml
```

There you have it. You should have a functioning ELK Stack deployment at
the end of the runs.

You will connect either to <http://logstashvipORlogstashvipname:5601> or
<http://logstashvipORlogstashvipname>.

Configure your devices to connect to the following (Unless you changed
the roles).

```yaml
logstash_inputs:
  - prot: tcp
    port: 10514
    type: syslog
  - prot: tcp
    port: 1514
    type: VMware
  - prot: tcp
    port: 1515
    type: vCenter
  - prot: tcp
    port: 1517
    type: Netscaler
  - prot: tcp
    port: 28778
    type: elasticsearch-curator
  - prot: tcp
    format: json
    port: 3515
    type: eventlog
  - prot: tcp
    codec: json_lines
    port: 3525
    type: iis
```

And of course if not on this list you will send to UDP/514 or whatever
else you may have added.
You will notice that VMware is listed above as tcp/1514. This worked
great until a few months back and messages began dropping on several
environments running ESXi 5.5U2+. To work around that we now configure
the ESXi hosts to send via udp/514 and all of the filtering is included
here to properly parse ESXi hosts.
For additional device setups you can still reference my original post
[here](https://everythingshouldbevirtual.com/highly-available-elk-elasticsearch-logstash-kibana-setup).

As I mentioned in the beginning, I am sure that this post will need some
modifications based on the fact that I have spent several months working
through all of this and moving the setup to Ansible. But also I have
made many changes in order to try and fit this into each unique
environment. However, I of course am just one person doing this with a
great understanding of how I have put this all together so I will need
to rely on each and everyone of your valuable input in order to make
this a better solution for all.

As always! Enjoy!
