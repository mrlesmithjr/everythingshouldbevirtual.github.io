---
  title: Ansible Playbook - GlusterFS - Apache
---

I wanted to share some of these playbooks for reference to others who
may be attempting to do something similar. Overall I am setting up some
Ubuntu servers which will be used for different functions (HAProxy load
balancers, MariaDB, Apache and GlusterFS). HAProxy will be load
balancing traffic back to the Apache servers and eventually MariaDB.
There are separate servers which will be running as GlusterFS servers in
which all of the Apache servers will be using a GlusterFS mount for
their /var/www to keep all servers in sync. I will be piecing together
more of this post as time goes on but for now I wanted to share the main
parts for reference. Feel free to leave comments on anything else that
might be of interest based on the information here.

Below is my site hosts inventory...(More on how this has been created
in the future :) )

`site_hosts`:

```bash
ans-test-db02  ansible_ssh_host=10.0.110.144 ansible_ssh_private_key_file=.ssh/home
ans-test-web03  ansible_ssh_host=10.0.110.130 ansible_ssh_private_key_file=.ssh/home
ans-test-web02  ansible_ssh_host=10.0.110.222 ansible_ssh_private_key_file=.ssh/home
ans-test-web01  ansible_ssh_host=10.0.110.193 ansible_ssh_private_key_file=.ssh/home
ans-test-db01  ansible_ssh_host=10.0.110.195 ansible_ssh_private_key_file=.ssh/home
ans-test-lb01  ansible_ssh_host=10.0.110.177 ansible_ssh_private_key_file=.ssh/home
ans-test-lb02  ansible_ssh_host=10.0.110.156 ansible_ssh_private_key_file=.ssh/home
ans-test-gs01  ansible_ssh_host=10.0.110.132 ansible_ssh_private_key_file=.ssh/home
ans-test-gs03  ansible_ssh_host=10.0.110.129 ansible_ssh_private_key_file=.ssh/home
ans-test-gs02  ansible_ssh_host=10.0.110.131 ansible_ssh_private_key_file=.ssh/home


[gs-vms]
ans-test-gs[01:03]

[gs-vms:vars]
config_lvm=true
create=true
create_vgname=glusterfs-vg
create_lvname=glusterfs-lv
new_mntp=/mnt/gluster
new_disk=/dev/sdb
filesystem=xfs
glusterfs_client=true
glusterfs_server=true
gluster_brick_dir={{ new_mntp }}
cluster_hosts='ans-test-gs01,ans-test-gs02,ans-test-gs03'

[web-vms]
ans-test-web[01:03]

[web-vms:vars]
glusterfs_client=true
primary_gfs_server=ans-test-gs01
secondary_gfs_server=ans-test-gs02
tertiary_gfs_server=ans-test-gs03
apache_backup_dir=/var/www.backup
apache_home=/var/www

[db-vms]
ans-test-db[01:02]

[db-vms:vars]
config_lvm=true
create=true
create_vgname=mysql-vg
create_lvname=mysql-lv
new_mntp=/var/lib/mysql
new_disk=/dev/sdb
filesytem=ext4
glusterfs_client=true

[lb-vms]
ans-test-lb01 keepalived_router_pri=101
ans-test-lb02 keepalived_router_pri=100

[lb-vms:vars]
config_haproxy=false
keepalived_vip=10.0.110.60
keepalived_vip_int=eth0
keepalived_router_id=51
glusterfs_client=true

### Below is for setting up lvm ---- these are the defaults in the base role


#config_lvm: false                                       # must be set to true in order to execute any tasks in play (failsafe option :)- )
#create: false                                           # set to true if creating a new logical volume (do not set extend or resize to true)
#resize: false                                           # set to true if resizing the logical volume (do not set create to true)
#extend: false                                           # set to true if extending the logical volume (do not set create to true)
#current_disk: '/dev/sda5'                               # set to your current disk device already setup in lvm
#new_disk: '/dev/sdb'                                    # set to new disk being added to volume group
#new_mntp: '/var/lib/mysql'                              # set to the desired mount point to be created and new logical volume to be mounted to
#create_vgname: 'test-vg'                               # set to volume group name to create
#resize_vgname: 'test-vg'                                # set to volume group name to resize
#extend_vgname: 'test-vg'                                # set to volume group name to extend
#create_lvname: 'test-lv'                               # set to logical volume name to create
#resize_lvname: 'test-lv'                                # set to logical volume name to resize
#extend_lvname: 'test-lv'                                # set to logical volume name to extend
#create_lvsize: '100%FREE'                               # set to logical volume size to create --- (10G) - would create new lvm with 10Gigabytes -- (512) - would create new lvm with 512m
#extend_disks: '{{ current_disk }},{{ new_disk }}'       # first disk is current volume group
#lvextend_options: '-L+10G'                              # setting the options to pass to lvextend --- ('-L+10G') - would increase by 10GB whereas ('-l +100%FREE') would increase to full capacity
#filesystem: 'ext4'                                      # set to filesystem type to format new logical volume with ( ext3, ext4, xfs, etc. )

#glusterfs_client=true|false
#glusterfs_server=true|false
```

Below is my site.yml file which includes some of the roles that I am
installing. I can post more about the roles or put up on github for
reference.

`site.yml`:

```yaml
---
- hosts: all
  sudo: yes
#  remote_user: home
  roles:
    - disable-firewall
#    - enable-firewall
    - { role: base, enable_cacti_monitoring: false }
    - zabbix-agent
#    - domain-join

#  vars_prompt:
#    - name: 'ad_user'
#      prompt: 'Enter domain user to join hosts to AD'
#      private: no
#    - name: 'ad_password'
#      prompt: 'Enter domain password to join hosts to AD'
#      private: yes

- hosts: web-vms
  roles:
#    - enable-firewall
    - glusterfs
    - apache2
    - memcached
    - logstash

- hosts: db-vms
  roles:
#    - enable-firewall
    - mariadb-mysql
    - logstash

- hosts: lb-vms
  roles:
#    - enable-firewall
    - glusterfs
    - keepalived
    - haproxy

- hosts: gs-vms
  roles:
#    - enable-firewall
     - glusterfs
```

Below is the playbook for the configuration of lvm.

`config_lvm.yml`:
{% raw %}

```yaml
---
- name: config_lvm | install | installing pre-reqs
  apt: name={{ item }} state=present
  with_items:
    - python-software-properties
    - xfsprogs
    - lvm2
  when: config_lvm and ansible_os_family == "Debian"

- name: config_lvm | install | installing lvm2
  yum: name=system-storage-manager state=present
  when: config_lvm and ansible_os_family == "RedHat"

- name: config_lvm | install | installing scsitools
  apt: name=scsitools state=present
  when: config_lvm and ansible_os_family == "Debian"

- name: config_lvm | install | installing sg3_utils
  yum: name=sg3_utils state=present
  when: config_lvm and ansible_os_family == "RedHat"

- name: config_lvm | config | rescanning for new disks
  command: /sbin/rescan-scsi-bus
  when: config_lvm and ansible_os_family == "Debian"

- name: config_lvm | config | rescanning for new disks
  command: /usr/bin/rescan-scsi-bus.sh
  when: config_lvm and ansible_os_family == "RedHat"

- name: config_lvm | config | creating new LVM volume group
  lvg: vg={{ create_vgname }} pvs={{ new_disk }} state=present
  when: create and config_lvm

- name: config_lvm | config | creating new LVM logical volume
  lvol: vg={{ create_vgname }} lv={{ create_lvname }} size={{ create_lvsize }}

- name: config_lvm | config | creating new filesystem on new LVM logical volume
  filesystem: fstype={{ filesystem }} dev=/dev/{{ create_vgname }}/{{ create_lvname }}
  when: create and config_lvm

- name: config_lvm | config | mounting new filesystem
  mount: name={{ new_mntp }} src=/dev/{{ create_vgname }}/{{ create_lvname }} fstype={{ filesystem }} state=mounted
  when: create and config_lvm

- name: config_lvm | config | extending existing LVM volume group
  lvg: vg={{ extend_vgname }} pvs={{ extend_disks }}
  when: extend and config_lvm

- name: config_lvm | config | extending existing filesystem
  command: lvextend {{ lvextend_options }} /dev/{{ extend_vgname }}/{{ extend_lvname }}
  when: extend and config_lvm

- name: config_lvm | config | resizing filesystem
  command: resize2fs /dev/{{ resize_vgname }}/{{ resize_lvname }}
  when: resize and config_lvm
```

{% endraw %}

Below is the playbook for the initial GlusterFS setup.

`debian_glusterfs.yml`:
{% raw %}

```yaml
---
- name: debian | install | installing pre-reqs
  apt: name={{ item }} state=present
  with_items:
    - python-software-properties
    - xfsprogs

- name: debian | config | adding glusterfs apt repo key
  apt_key: keyserver=keyserver.ubuntu.com id=F7C73FCC930AC9F83B387A5613E01B7B3FE869A9 state=present

- name: debian | config | adding glusterfs apt repo
  apt_repository: repo='deb http://ppa.launchpad.net/gluster/glusterfs-{{glusterfs_version}}/ubuntu trusty main' state=present update_cache=yes

- name: debian | install | installing glusterfs server
  apt: name=glusterfs-server state=latest
  when: glusterfs_server

- name: debian | install | installing glusterfs client
  apt: name=glusterfs-client state=latest
  when: glusterfs_client

- name: debian | config | enabling glusterfs-server
  service: name=glusterfs-server enabled=yes
  when: glusterfs_server
```

{% endraw %}

And finally the actual GlusterFS playbook to do the actual
configurations on the web servers and GlusterFS servers.

`glusterfs.yml`:
{% raw %}

```yaml
---
- hosts: gs-vms
  tasks:
    - name: Start GlusterFS
      service: name=glusterfs-server state=started enabled=true

    - name: connect gluster peers
      command: gluster peer probe {{ item }}
      register: gluster_peer_probe
      changed_when: "'already in peer list' not in gluster_peer_probe.stdout"
      with_items: groups['gs-vms']

    - name: creating brick folders
      file: path={{ gluster_brick_dir }}/{{ item.name }}/ owner={{ item.owner }} group={{ item.group }} state=directory
      with_items:
        - { name: loadbalancers, owner: root, group: root }
        - { name: webs, owner: root, group: www-data }

    - name: create gluster volume
      gluster_volume: state=present name={{ item.name }} brick={{ item.brick }} rebalance=yes replicas=3 cluster={{ cluster_hosts }}
      with_items:
        - { name: webs-test, brick: /mnt/gluster/webs/test }
        - { name: loadbalancers-test, brick: /mnt/gluster/loadbalancers/test }
      ignore_errors: yes
      run_once: true

- hosts: web-vms
  tasks:
    - name: checking to see if /var/www has already been moved
      stat: path={{ apache_backup_dir }}
      register: www_backup_moved

    - name: moving existing /var/www
      command: mv {{ apache_home }} {{ apache_backup_dir }}
      when: www_backup_moved.stat.exists != true

    - name: checking again if /var/www has already been moved
      stat: path={{ apache_backup_dir }}
      register: www_backup_stat

    - name: touching file in apache_backup_dir
      file: path={{ apache_backup_dir }}/already_moved state=touch
      when: www_backup_stat.stat.exists == true

    - name: mounting gluster volumes
      mount: name={{ item.mountpoint }} src={{ item.src }} fstype={{ item.fstype }} opts={{ item.options }} state=mounted
      with_items:
        - { mountpoint: '{{ apache_home }}', src: '{{ primary_gfs_server }}:/webs-test', fstype: 'glusterfs', options: 'defaults,_netdev,backupvolfile-server={{ secondary_gfs_server }},backupvolfile-server={{ tertiary_gfs_server }}' }
      when: www_backup_stat.stat.exists == true
```

{% endraw %}
