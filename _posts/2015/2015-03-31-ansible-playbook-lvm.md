---
  title: Ansible Playbook - LVM
---

In spending the past few weeks learning a ton about Ansible (after
creating 30+ playbooks and roles - Some very complex :)  more goodness
coming soon ).. One thing I wanted to do with Ansible was to configure
LVM on my Linux hosts. After some digging around and testing I came up
with this playbook and figured I would share it with others in case they
had a need for it as well. So here it is.. Will also be updating and
adding a bit more functionality around it as well. All you need to do is
tweak some of the variables to fit your requirements.

{% raw %}

```yaml
---
- hosts: db-vms                                             # set to specific inventory host group or set to all for every host in inventory for play
  vars:
    config_lvm: false                                       # must be set to true in order to execute any tasks in play (failsafe option :)- )
    create: false                                           # set to true if creating a new logical volume (do not set extend or resize to true)
    resize: false                                           # set to true if resizing the logical volume (do not set create to true)
    extend: false                                           # set to true if extending the logical volume (do not set create to true)
    current_disk: '/dev/sda5'                               # set to your current disk device already setup in lvm
    new_disk: '/dev/sdb'                                    # set to new disk being added to volume group
    new_mntp: '/var/lib/mysql'                              # set to the desired mount point to be created and new logical volume to be mounted to
    create_vgname: 'mysql-vg'                               # set to volume group name to create
    resize_vgname: 'test-vg'                                # set to volume group name to resize
    extend_vgname: 'test-vg'                                # set to volume group name to extend
    create_lvname: 'mysql-lv'                               # set to logical volume name to create
    resize_lvname: 'test-lv'                                # set to logical volume name to resize
    extend_lvname: 'test-lv'                                # set to logical volume name to extend
    create_lvsize: '100%FREE'                               # set to logical volume size to create --- (10G) - would create new lvm with 10Gigabytes -- (512) - would create new lvm with 512m
    extend_disks: '{{ current_disk }},{{ new_disk }}'       # first disk is current volume group
    lvextend_options: '-L+10G'                              # setting the options to pass to lvextend --- ('-L+10G') - would increase by 10GB whereas ('-l +100%FREE') would increase to full capacity
    filesystem: 'ext4'                                      # set to filesystem type to format new logical volume with ( ext3, ext4, xfs, etc. )
  tasks:
    - name: installing lvm2
      apt: name=lvm2 state=present
      when: config_lvm and ansible_os_family == "Debian"

    - name: installing lvm2
      yum: name=system-storage-manager state=present
      when: config_lvm and ansible_os_family == "RedHat"

    - name: installing scsitools
      apt: name=scsitools state=present
      when: config_lvm and ansible_os_family == "Debian"

    - name: installing sg3_utils
      yum: name=sg3_utils state=present
      when: config_lvm and ansible_os_family == "RedHat"

    - name: rescanning for new disks
      command: /sbin/rescan-scsi-bus
      when: config_lvm and ansible_os_family == "Debian"

    - name: rescanning for new disks
      command: /usr/bin/rescan-scsi-bus.sh
      when: config_lvm and ansible_os_family == "RedHat"

    - name: creating new LVM volume group
      lvg: vg={{ create_vgname }} pvs={{ new_disk }} state=present
      when: create and config_lvm

    - name: creating new LVM logical volume
      lvol: vg={{ create_vgname }} lv={{ create_lvname }} size={{ create_lvsize }}
      when: create and config_lvm

    - name: creating new filesystem on new LVM logical volume
      filesystem: fstype={{ filesystem }} dev=/dev/{{ create_vgname }}/{{ create_lvname }}
      when: create and config_lvm

    - name: mounting new filesystem
      mount: name={{ new_mntp }} src=/dev/{{ create_vgname }}/{{ create_lvname }} fstype={{ filesystem }} state=mounted
      when: create and config_lvm

    - name: extending existing LVM volume group
      lvg: vg={{ extend_vgname }} pvs={{ extend_disks }}
      when: extend and config_lvm

    - name: extending existing filesystem
      command: lvextend {{ lvextend_options }} /dev/{{ extend_vgname }}/{{ extend_lvname }}
      when: extend and config_lvm

    - name: resizing filesystem
      command: resize2fs /dev/{{ resize_vgname }}/{{ resize_lvname }}
      when: resize and config_lvm
```

{% endraw %}

Enjoy!
