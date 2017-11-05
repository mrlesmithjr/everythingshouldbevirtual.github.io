---
  title: Ansible - Installing open-vm-tools
---

Just sharing this out in case someone else may be looking for a way to
install open-vm-tools for Ubuntu or CentOS and wants to ensure that only
virtual machines get this package installed.

Throw together a playbook or role and add the following bit of code and
you are good to go.

```yaml
- name: debian | installing open-vm-tools
  apt: name=open-vm-tools state=present
  when: ansible_os_family == "Debian" and ansible_virtualization_type == "VMware"

- name: centos | installing open-vm-tools if a vm
  yum: name=open-vm-tools state=present
  when: ansible_os_family == "RedHat" and ansible_virtualization_type == "VMware"

- name: centos | starting and enabling open-vm-tools
  service: name=vmtoolsd.service state=restarted enabled=yes
  when: ansible_os_family == "RedHat" and ansible_virtualization_type == "VMware"
```

There you have it. Now when you run your playbook or roles open-vm-tools
will be installed on only devices that are VMware virtual machines.

Enjoy!
