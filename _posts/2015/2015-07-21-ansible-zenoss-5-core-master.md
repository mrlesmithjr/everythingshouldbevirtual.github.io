---
  title: Ansible - Zenoss 5 Core - Master
---

As it has been some time since actually putting something together to
install [Zenoss](http://www.zenoss.org/) I figured I would set out to do
just that using [Ansible](http://ansible.com). As I began through this
it seems that the installation process is much more involved than
previous versions. First off, the majority of Zenoss 5 is container
based, i.e. [Docker](http://docker.com). So this is a good thing in
regards to upgrades and etc. However the requirements are extreme.

## Zenoss Master

[docs](http://www.zenoss.com/sites/default/files/documentation/Zenoss_Resource_Manager_Installation_Guide_r5.0.x_d1052.15.196.pdf)

-   8 CPU cores (64-bit only; real or virtual)
-   32GB RAM
-   / xfs 90GB Local disk. Includes space for internal services data and
    some backups.
-   (none) swap 15GB Local disk.
-   /var/lib/docker xfs 60GB Local disk.
-   /opt/serviced/var/volumes btrfs 1TB Remote SAN.

So just to begin this journey I wanted to share out the playbook to
deploy the Zenoss Core master server (additional playbooks coming soon).

First thing first is to deploy a server for this. This is based on
**_CentOS 7 x64_**. I was able to deploy using a **_4CPU_** and **_8GB
RAM_** vm but it is painfully slow.

Add the following hard drives to your vm when deploying. Do not
partition SDB and SDC as these will be partitioned, formatted and
mounted using the playbook.

-   SDA - / (Root)  36GB+
-   SDB - docker partition 50GB+
-   SDC - app partition 100GB+

Now that your server is deployed you can run the following playbook to
setup and configure your Zenoss 5 Core Master server. Feel free to make
changes where you deem necessary. One being the administrator
password... The password hashed in the playbook is `P@55w0rd`
{% raw %}

```yaml
---
- hosts: zenoss-master
  remote_user: remote
  sudo: true
  vars:
    packages:
      - dnsmasq
      - nfs-utils
      - ntp
    services:
      - dnsmasq
      - rpcbind
      - nfs-server
      - nfs-lock
      - nfs-idmap
      - ntpd
    zenoss_btrfs_mounts:
      - /var/lib/docker
      - /opt/serviced/var/volumes
    zenoss_disks:
      - /dev/sdb
      - /dev/sdc
    zenoss_file_systems:
      - name: docker_part
        mount: /var/lib/docker
        device: /dev/sdb1
      - name: app_part
        mount: /opt/serviced/var/volumes
        device: /dev/sdc1
    zenoss_package: http://get.zenoss.io/yum/zenoss-repo-1-1.x86_64.rpm
    zenoss_users:
      - name: administrator
        password: $6$I.7NKW9uP$MPB.Gpisa.sfpb//pSSufZm03.kMkQ9YhAJVjvFtpBANKgXfmhV2YTySULVN9VDrfG1tvP3S2pVWaRLja6XeN/
        groups: wheel
  tasks:
    - name: stopping firewall and disabling firewall
      service: name=firewalld state=stopped enabled=false

    - name: checking for journal log
      stat: path=/var/log/journal
      register: journal_log

    - name: creating journal log
      file: path=/var/log/journal state=directory
      register: journal_log_created
      when: not journal_log.stat.exists

    - name: restarting journal
      service: name=systemd-journald state=restarted
      when: journal_log_created.changed

    - name: checking for existing partioned disks
      stat: path=/var/log/.partitions_created
      register: partitions_exist

    - name: partitioning disks
      shell: "echo -e 'n\np\n1\n\n\nw\n' | fdisk {{ item }}"
      register: partitions_created
      with_items: zenoss_disks
      when: not partitions_exist.stat.exists

    - name: marking as partions created
      file: path=/var/log/.partitions_created state=touch
      when: partitions_created.changed

    - name: creating filesystems
      filesystem: fstype=btrfs dev={{ item.device }}
      with_items: zenoss_file_systems

    - name: mounting filesystems
      mount: name={{ item.mount }} src={{ item.device }} fstype=btrfs opts=noatime,nodatacow state=mounted
      with_items: zenoss_file_systems

    - name: disabling selinux
      selinux: state=disabled
      register: disabled_selinux

    - name: installing zenoss repo
      yum: name={{ zenoss_package }} state=present

    - name: installing packages
      yum: name={{ item }} state=present
      with_items: packages

    - name: starting services and enabling services
      service: name={{ item }} state=started enabled=true
      with_items: services

    - name: restarting server to complete disabling selinux
      command: shutdown -r now "rebooting to complete disabling selinux"
      async: 0
      poll: 0
      ignore_errors: true
      when: disabled_selinux.changed

    - name: waiting for server to come back
      local_action: wait_for host="{{ ansible_ssh_host  | default(inventory_hostname) }}" state=started
      sudo: false
      when: disabled_selinux.changed

    - name: installing zenoss master (Control Center, Zenoss Core and Docker)
      yum: name=zenoss-core-service enablerepo=zenoss-stable

    - name: starting docker
      service: name=docker state=started

    - name: capturing docker IP address
      shell: "ip addr ls docker0 | awk '/inet / {print $2}' | cut -d\"/\" -f1"
      register: docker_ip

    - name: adding btrfs and DNS flags to docker startup
      lineinfile: dest=/etc/sysconfig/docker regexp="^DOCKER_OPTS" line="DOCKER_OPTS=\"-s btrfs --dns={{ docker_ip.stdout }}\""
      register: docker_reconfigured

    - name: adding root user to docker group
      user: name=root groups=docker append=yes

    - name: restarting docker
      service: name=docker state=restarted
      when: docker_reconfigured.changed

    - name: changing SERVICED_FS_TYPE to btrfs
      replace: dest=/etc/default/serviced regexp="# SERVICED_FS_TYPE=rsync" replace="SERVICED_FS_TYPE=btrfs"

    - name: starting service center
      service: name=serviced state=started

    - name: creating and allowing access for users to control center
      user: name={{ item.name }} groups={{ item.groups }} password={{ item.password }} state=present append=yes
      with_items: zenoss_users

    - name: restarting service center
      service: name=serviced state=restarted
```

{% endraw %}
If all goes well you should be able to connect to
**_<https://IPorHostname>_** and login with administrator and whatever
password you have defined in the playbook. Once logged in you can follow
the setup wizard to finalize the setup of your new Zenoss 5 Core Master
node.

That's all for now but I will be publishing the additional roles soon
to complete out the install of Zenoss Core.

Enjoy!
