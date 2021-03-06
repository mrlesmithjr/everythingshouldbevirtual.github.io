---
  title: Ansible - Quick Inventory Gathering using loops
---

I came across this today looking through the Ansible Google groups. The
OP wanted something that would output something like below into separate
files per inventory hostname.

```bash
Linux ans-test-4 3.13.0-55-generic #92-Ubuntu SMP Sun Jun 14 18:32:20 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 192.168.70.240
nameserver 192.168.70.241
search everythingshouldbevirtual.local
```

The OP was close to having the solution but after a bit of throwing some
stuff together below is what ends up getting us what we are looking for.
Obviously there are many other ways to pull these sorts of information
but trying to stay close to the original question this is what worked.
{% raw %}

```yaml
---
- hosts: all
  gather_facts: true
  remote_user: remote
  sudo: true
  tasks:
    - name: capture linux_os
      shell: "{{ item }}"
      register: linux_os
      with_items:
        - 'uname -a'
        - 'cat /etc/resolv.conf'

    - name: creating inventory hosts
      file: path="{{ inventory_hostname }}".txt state=touch
      delegate_to: localhost
      sudo: false

    - name: results
      lineinfile: dest="{{ inventory_hostname }}".txt regexp="^{{ item.stdout }}" line="{{ item.stdout }}"
      delegate_to: localhost
      sudo: false
      with_items: linux_os.results
```

{% endraw %}
Enjoy!
