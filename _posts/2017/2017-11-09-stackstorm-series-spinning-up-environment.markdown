---
title: "StackStorm Series - Spinning Up Environment"
date: "2017-11-09 00:50"
categories:
  - Series
tags:
  - StackStorm
---

This post is the next in the StackStorm Series. You can find the previous posts
in the series below:

-   [StackStorm Series - Intro](https://everythingshouldbevirtual.com/series/stackstorm-series-intro/)
-   [StackStorm Series - Getting Started](https://everythingshouldbevirtual.com/series/stackstorm-series-getting-started/)

In this post we will be spinning up the test environment for us to begin our
StackStorm learning.

> NOTE: Assumption is that you have already followed the steps in the previous
> posts.

## Spinning Up Environment

So let's get to it and spin the StackStorm environment up. Because we are using
[Vagrant](https://www.vagrantup.com/) the steps to spin everything up is as
simple as a single command:

> NOTE: Ensure that you are in your project folder and within the `Vagrant` folder.

```bash
vagrant up
```

Now sit back and grab a coffee or something while the environment is provisioned.
The whole provisioning takes about ~10 minutes to complete.

If you decide to watch then you should see everything spinning up as below:

```raw
Bringing machine 'node0' up with 'virtualbox' provider...
Bringing machine 'node1' up with 'virtualbox' provider...
==> node0: Importing base box 'mrlesmithjr/xenial64'...
==> node0: Matching MAC address for NAT networking...
==> node0: Checking if box 'mrlesmithjr/xenial64' is up to date...
==> node0: Setting the name of the VM: Vagrant_node0_1510207057205_94918
==> node0: Clearing any previously set network interfaces...
==> node0: Preparing network interfaces based on configuration...
    node0: Adapter 1: nat
    node0: Adapter 2: hostonly
==> node0: Forwarding ports...
    node0: 22 (guest) => 2222 (host) (adapter 1)
```

And then you will see Ansible doing it's thing as well:

```raw
    node1: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [node0]
ok: [node1]

TASK [updating apt cache (Debian)] *********************************************
ok: [node0]
ok: [node1]

TASK [installing ansible pre-reqs (Debian)] ************************************
changed: [node0] => (item=[u'build-essential', u'libffi-dev', u'libssl-dev', u'python-dev', u'python-setuptools'])
changed: [node1] => (item=[u'build-essential', u'libffi-dev', u'libssl-dev', u'python-dev', u'python-setuptools'])

TASK [installing epel repo (RedHat)] *******************************************
skipping: [node0]
skipping: [node1]

TASK [installing ansible pre-reqs (RedHat)] ************************************
skipping: [node0] => (item=[])
skipping: [node1] => (item=[])
```

And once the environment is provisioned you should see the end of the Ansible
provisioning:

```raw
RUNNING HANDLER [ansible-nginx : restart nginx] ********************************
changed: [node0]

RUNNING HANDLER [ansible-nginx : restart php7.0-fpm] ***************************
changed: [node0]

RUNNING HANDLER [ansible-rabbitmq : restart rabbitmq-server] *******************
changed: [node0]

RUNNING HANDLER [ansible-stackstorm : reload stackstorm] ***********************
changed: [node0]

RUNNING HANDLER [ansible-stackstorm : restart stackstorm] **********************
changed: [node0]

PLAY [stackstorm_client] *******************************************************

TASK [Gathering Facts] *********************************************************
ok: [node1]

TASK [ansible-users : users | managing user accounts] **************************
changed: [node1] => (item={u'comment': u'Stackstorm SSH User', u'generate_keys': True, u'preseed_user': False, u'sudo': True, u'state': u'present', u'user': u'stanley', u'pass': u'$6$8tMUxKP33/$Fb/hZBaYvyzGubO9nrlRJMjUnt3aajXZwxCifH9NYqrhjMlC9COWmNNFiMpnyNGsgmDeNCCn2wKNh0G1E1BBV0', u'system_account': False})

TASK [ansible-users : users | adding users to sudoers] *************************
changed: [node1] => (item={u'comment': u'Stackstorm SSH User', u'generate_keys': True, u'preseed_user': False, u'sudo': True, u'state': u'present', u'user': u'stanley', u'pass': u'$6$8tMUxKP33/$Fb/hZBaYvyzGubO9nrlRJMjUnt3aajXZwxCifH9NYqrhjMlC9COWmNNFiMpnyNGsgmDeNCCn2wKNh0G1E1BBV0', u'system_account': False})

TASK [ansible-users : users | removing users from sudoers] *********************
skipping: [node1] => (item={u'comment': u'Stackstorm SSH User', u'generate_keys': True, u'preseed_user': False, u'sudo': True, u'state': u'present', u'user': u'stanley', u'pass': u'$6$8tMUxKP33/$Fb/hZBaYvyzGubO9nrlRJMjUnt3aajXZwxCifH9NYqrhjMlC9COWmNNFiMpnyNGsgmDeNCCn2wKNh0G1E1BBV0', u'system_account': False})

TASK [ansible-manage-ssh-keys : manage_ssh_keys | managing ssh user keys] ******
changed: [node1] => (item=({u'state': u'present', u'remote_user': u'stanley'}, u'./stanley@node0.pub'))

PLAY RECAP *********************************************************************
node0                      : ok=64   changed=51   unreachable=0    failed=0
node1                      : ok=6    changed=4    unreachable=0    failed=0
```

So there you have it, a fully provisioned and ready StackStorm environment. We are
now ready for the next post of the series where we will begin to explore the WebUI.

Enjoy!
