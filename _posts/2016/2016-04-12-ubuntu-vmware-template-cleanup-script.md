---
  title: Ubuntu - VMware Template Cleanup Script
  categories:
    - Virtualization
  tags:
    - VMware
---

Just sharing these scripts in case anyone ever needs/wants a quick way
to prep an Ubuntu VM template with/without guest customizations. One
script is just a shell script and the other is an Ansible play that can
be used (Ansible Must be installed in guest).

```bash
#!/bin/bash

# Add usernames to add to /etc/sudoers for passwordless sudo
users=("ubuntu" "admin")

for user in "${users[@]}"
do
  cat /etc/sudoers | grep ^$user
  RC=$?
  if [ $RC != 0 ]; then
    bash -c "echo \"$user ALL=(ALL) NOPASSWD:ALL\" >> /etc/sudoers"
  fi
done

#add VMware package keys
# wget http://packages.vmware.com/tools/keys/VMWARE-PACKAGING-GPG-DSA-KEY.pub -O - | apt-key add -
# wget http://packages.vmware.com/tools/keys/VMWARE-PACKAGING-GPG-RSA-KEY.pub -O - | apt-key add -

#grab Ubuntu Codename
codename="$(lsb_release -c | awk {'print $2}')"

#add VMware repository to install open-vm-tools-deploypkg
# echo "deb http://packages.vmware.com/packages/ubuntu $codename main" > /etc/apt/sources.list.d/vmware-tools.list

#update apt-cache
apt-get update

#install packages
# apt-get install -y open-vm-tools
apt-get install -y python-minimal

#Stop services for cleanup
service rsyslog stop

#clear audit logs
if [ -f /var/log/audit/audit.log ]; then
    cat /dev/null > /var/log/audit/audit.log
fi
if [ -f /var/log/wtmp ]; then
    cat /dev/null > /var/log/wtmp
fi
if [ -f /var/log/lastlog ]; then
    cat /dev/null > /var/log/lastlog
fi

#cleanup persistent udev rules
if [ -f /etc/udev/rules.d/70-persistent-net.rules ]; then
    rm /etc/udev/rules.d/70-persistent-net.rules
fi

#cleanup /tmp directories
rm -rf /tmp/*
rm -rf /var/tmp/*

#cleanup current ssh keys
rm -f /etc/ssh/ssh_host_*

#add check for ssh keys on reboot...regenerate if neccessary
sed -i -e 's|exit 0||' /etc/rc.local
sed -i -e 's|.*test -f /etc/ssh/ssh_host_dsa_key.*||' /etc/rc.local
bash -c 'echo "test -f /etc/ssh/ssh_host_dsa_key || dpkg-reconfigure openssh-server" >> /etc/rc.local'
bash -c 'echo "exit 0" >> /etc/rc.local'

#reset hostname
cat /dev/null > /etc/hostname

#cleanup apt
apt-get clean

#cleanup shell history
history -w
history -c
```

{% raw %}

```yaml
---
- hosts: all
  become: true
  vars:
    - audit_logs:
        - '/var/log/audit/audit.log'
        - '/var/log/wtmp'
        - '/var/log/lastlog'
  tasks:
    - name: adding VMware repo keys
      apt_key:
        url: "{{ item }}"
        state: present
      with_items:
        - 'http://packages.vmware.com/tools/keys/VMWARE-PACKAGING-GPG-DSA-KEY.pub'
        - 'http://packages.vmware.com/tools/keys/VMWARE-PACKAGING-GPG-RSA-KEY.pub'
      when: ansible_os_family == "Debian"

    - name: adding VMware repository
      apt_repository:
        repo: "deb http://packages.vmware.com/packages/ubuntu {{ ansible_distribution_release }} main"
        state: present
      register: repo_added
      when: >
            ansible_distribution == "Ubuntu" and
            (ansible_distribution_release == "precise" or
            ansible_distribution_release == "trusty")

    - name: updating apt-cache
      apt:
        update_cache: true
      when: >
            ansible_distribution == "Ubuntu" and
            repo_added.changed

    - name: installing open-vm-tools-deploypkg
      apt:
        name: "open-vm-tools-deploypkg"
        state: present
      when: >
            ansible_distribution == "Ubuntu" and
            (ansible_distribution_release == "precise" or
            ansible_distribution_release == "trusty")

    - name: installing open-vm-tools
      apt:
        name: "open-vm-tools"
        state: present
      when: ansible_os_family == "Debian"

    - name: stopping rsyslog service
      service:
        name: "rsyslog"
        state: stopped
      when: ansible_os_family == "Debian"

    - name: clearing audit logs
      file:
        path: "{{ item }}"
        state: absent
      with_items: audit_logs

    - name: cleaning up udev rules
      file:
        path: "/etc/udev/rules.d/70-persistent-net.rules"
        state: absent

    - name: cleaning up tmp directories
      command: "rm -rf {{ item }}"
      with_items:
        - '/tmp/*'
        - '/var/tmp/*'

    - name: cleaning up ssh keys
      command: "rm -f /etc/ssh/ssh_host_*"
```

{% endraw %}

Enjoy!
