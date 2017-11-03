---
  title: Ansible - SSH Known Host Keys
  categories:
    - Automation
  tags:
    - Ansible
---

I wanted to throw this together mainly for my own reference but maybe it
will help someone else as well. I had a need to add every host's ssh
keys to every host so that every host knew what every other hosts ssh
keys were. After a bit of attempting many different things below is what
I came up with. And it works.

First create a simple playbook:

{% gist mrlesmithjr/71f0669dfb970b4904a7cbe8b8e46863 %}

Next create this simple template:

{% gist mrlesmithjr/10a0ad5ef831ca83e28f9b100e0f8ac6 %}

Then run

```bash
ansible-playbook -i yourinventoryfile ssh-keys.yml
```

and it will run through each host and capture their respective ssh key
and then create /etc/ssh/ssh_known_hosts on each host including all
other hosts ssh keys as well. Pretty simple after quite a bit of trial
and error but it does work.

Enjoy!
