---
  title: Ansible - Clean Formatted Playbooks
  categories:
    - Automation
  tags:
    - Ansible
  redirect_from:
    - /ansible-clean-formatted-playbooks
---

While going through and doing some cleanup to various different roles
and playbooks in my Ansible collection I wanted to share what I feel is
good clean formatting. This will of course not be for everyone but I
wanted to share this nonetheless. The goal of writing playbooks is clean
formatted tasks and such to make them easily readable to other or even
yourself when going back to look your playbooks. One thing I have
noticed over time is that not everyone follows this and sometimes it is
extremely difficult to understand what is going on and especially so for
those who are just learning. So below you will find an example of an
Ansible playbook that is running many different plays and utilizing
roles throughout. Feel free to leave your thoughts so others may learn
from this as well.

```yaml
---
- name: Configure DDI Nodes
  hosts: ddi-nodes
  become: true
#  remote_user: remote
  roles:
    - role: ansible-manage-lvm
      when: manage_lvm is defined and manage_lvm
      tags:
        - manage_lvm
    - role: ansible-config-interfaces
      tags:
        - config_interfaces
    - role: ansible-apache2
    - role: ansible-ntp
      tags:
        - config_ntp
    - role: ansible-mariadb-galera-cluster
    - role: ansible-logstash
      when: install_logstash is defined and install_logstash

- name: Configure Quagga on DDI Nameserver Nodes
  hosts: ddi-nameserver-nodes
  become: true
#  remote_user: remote
  roles:
    - role: ansible-quagga
      when: >
            (install_quagga is defined and install_quagga) and
            (enable_pdns_anycast is defined and enable_pdns_anycast)

- name: Configure DDI Services on DDI Nameserver Nodes
  hosts: ddi-nameserver-nodes
  become: true
#  remote_user: remote
  tasks:
    - name: debian | uninstalling ISC-DHCP
      apt:
        name: isc-dhcp-server
        state: absent
      when: install_dhcp is defined and not install_dhcp
  roles:
    - role: ansible-powerdns
      when: install_dns is defined and install_dns
    - role: ansible-phpipam
      when: install_phpipam is defined and install_phpipam
    - role: ansible-isc-dhcp
      tags:
        - config_dhcp
      when: install_dhcp is defined and install_dhcp

- name: Configure DDI Services on DDI Nameserver RO Nodes
  hosts: ddi-nameserver-ro-nodes
  become: true
#  remote_user: remote
  tasks:
  roles:
    - role: ansible-config-interfaces
      tags:
        - config_interfaces
    - role: ansible-logstash
      when: install_logstash is defined and install_logstash
    - role: ansible-ntp
      tags:
        - config_ntp
    - role: ansible-powerdns
      when: install_dns is defined and install_dns
```

As you can see from above the when conditions are being derived from
either the defaults in the roles themselves or from group_vars and/or
host_vars. Also note the when conditions highlighted and see how we can
write them differently. The second one is a good way to format a clean
extended when condition so the whole when condition is not across the
whole screen (I am guilty of this one as well).

How about another way to define these differently then using external
var definitions? In the example below you will see how we can define
within our playbook on how to define vars for specific roles within each
of our plays in our playbook.

```yaml
---
- name: Configure DDI Nodes
  hosts: ddi-nodes
  become: true
#  remote_user: remote
  roles:
    - role: ansible-manage-lvm
      manage_lvm: true
      when: manage_lvm is defined and manage_lvm
      tags:
        - manage_lvm
    - role: ansible-config-interfaces
      tags:
        - config_interfaces
    - role: ansible-apache2
    - role: ansible-ntp
      tags:
        - config_ntp
    - role: ansible-mariadb-galera-cluster
    - role: ansible-logstash
      install_logstash: false
      when: install_logstash is defined and install_logstash

- name: Configure Quagga on DDI Nameserver Nodes
  hosts: ddi-nameserver-nodes
  become: true
#  remote_user: remote
  roles:
    - role: ansible-quagga
      install_quagga: false
      enable_pdns_anycast: false
      when: >
            (install_quagga is defined and install_quagga) and
            (enable_pdns_anycast is defined and enable_pdns_anycast)

- name: Configure DDI Services on DDI Nameserver Nodes
  hosts: ddi-nameserver-nodes
  become: true
#  remote_user: remote
  tasks:
    - name: debian | uninstalling ISC-DHCP
      apt:
        name: isc-dhcp-server
        state: absent
      when: install_dhcp is defined and not install_dhcp
  roles:
    - role: ansible-powerdns
      install_dns: true
      when: install_dns is defined and install_dns
    - role: ansible-phpipam
      install_phpipam: true
      when: install_phpipam is defined and install_phpipam
    - role: ansible-isc-dhcp
      install_dhcp: true
      tags:
        - config_dhcp
      when: install_dhcp is defined and install_dhcp

- name: Configure DDI Services on DDI Nameserver RO Nodes
  hosts: ddi-nameserver-ro-nodes
  become: true
#  remote_user: remote
  tasks:
  roles:
    - role: ansible-config-interfaces
      tags:
        - config_interfaces
    - role: ansible-logstash
      install_logstash: true
      when: install_logstash is defined and install_logstash
    - role: ansible-ntp
      tags:
        - config_ntp
    - role: ansible-powerdns
      install_dns: true
      when: install_dns is defined and install_dns
```

As you can see from the highlighted lines dictate what our vars should
be set to for that specific play against its respective role.

And yet one other example of defining variables for a whole play to be
applied against all roles.

```yaml
---
- name: Configure DDI Nodes
  hosts: ddi-nodes
  become: true
#  remote_user: remote
  vars:
    install_logstash: false
    manage_lvm: true
  roles:
    - role: ansible-manage-lvm
      when: manage_lvm is defined and manage_lvm
      tags:
        - manage_lvm
    - role: ansible-config-interfaces
      tags:
        - config_interfaces
    - role: ansible-apache2
    - role: ansible-ntp
      tags:
        - config_ntp
    - role: ansible-mariadb-galera-cluster
    - role: ansible-logstash
      when: install_logstash is defined and install_logstash

- name: Configure Quagga on DDI Nameserver Nodes
  hosts: ddi-nameserver-nodes
  become: true
#  remote_user: remote
  vars:
    enable_pdns_anycast: false
    install_quagga: false
  roles:
    - role: ansible-quagga
      when: >
            (install_quagga is defined and install_quagga) and
            (enable_pdns_anycast is defined and enable_pdns_anycast)

- name: Configure DDI Services on DDI Nameserver Nodes
  hosts: ddi-nameserver-nodes
  become: true
#  remote_user: remote
  vars:
    install_dhcp: true
    install_dns: true
    install_phpipam: true
    install_dhcp: true
  tasks:
    - name: debian | uninstalling ISC-DHCP
      apt:
        name: isc-dhcp-server
        state: absent
      when: install_dhcp is defined and not install_dhcp
  roles:
    - role: ansible-powerdns
      when: install_dns is defined and install_dns
    - role: ansible-phpipam
      when: install_phpipam is defined and install_phpipam
    - role: ansible-isc-dhcp
      tags:
        - config_dhcp
      when: install_dhcp is defined and install_dhcp

- name: Configure DDI Services on DDI Nameserver RO Nodes
  hosts: ddi-nameserver-ro-nodes
  become: true
#  remote_user: remote
  vars:
    install_dns: true
    install_logstash: true
  tasks:
  roles:
    - role: ansible-config-interfaces
      tags:
        - config_interfaces
    - role: ansible-logstash
      when: install_logstash is defined and install_logstash
    - role: ansible-ntp
      tags:
        - config_ntp
    - role: ansible-powerdns
      when: install_dns is defined and install_dns
```

As you can see from above we have defined all of our vars outside of the
roles and within the actual play details.

The above are only a few examples of what I would consider writing good
clean Ansible playbooks, tasks and etc.

One last item I would like to share is when writing Ansible handlers.
One may assume that handlers can/or only look like the example below.

```yaml
- name: Example Handler Playbook
  hosts: all
  become: true
  vars:
    are_you_sure: false
    reconfigure_apache2: true
  handlers:
    - name: restart apache2
      service:
        name: apache2
        state: restarted
      when: are_you_sure is defined and are_you_sure
  tasks:
    - name: Reconfigure Apache2
      template:
        src: "etc/apache2/apache2.conf.j2"
        dest: "/etc/apache2/apache2/conf"
        owner: root
        group: root
        mode: 0644
      notify:
        - restart apache2
      when: reconfigure_apache2 is defined and reconfigure_apache2
```

What is being shown above is that we can add a when condition to a
handler as well. This short play would reconfigure Apache2 because we
have **reconfigure_apache2: true **but the handler defined to notify
the restart of apache2 will not occur because we have the variable
**are_you_sure: false**.

This is all for now and hope this may find to be useful to others as
well as encourage others to share their thoughts or even tips and
tricks.

Enjoy!
