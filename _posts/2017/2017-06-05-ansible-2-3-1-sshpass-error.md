---
  title: Ansible 2.3.1 - sshpass Error
  categories:
    - Automation
  tags:
    - Ansible
---

As you may or may not be aware, **Ansible 2.3.1.0** was recently
released. After installing **Ansible 2.3.1.0** in my Python virtual
environment I immediately ran into an issue that I had not seen. My host
machine that I am running Ansible from is a MacBook Pro running Sierra
so YMMV. Of course once I saw this I immediately activated my **Ansible
2.3.0.0** environment and sure enough the issue was not present.

```bash
ansible-playbook -i inventory playbooks/kvm.yml --vault-password-file ~/.vault_pass --user remote

PLAY [kvm_hosts] *****************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
fatal: [ms-7693.etsbv.internal]: FAILED! => {"failed": true, "msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [nas01.etsbv.internal]: FAILED! => {"failed": true, "msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [nas02.etsbv.internal]: FAILED! => {"failed": true, "msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
    to retry, use: --limit @./.retry/kvm.retry

PLAY RECAP ***********************************************************************************************************************************************************************
ms-7693.etsbv.internal     : ok=0    changed=0    unreachable=0    failed=1
nas01.etsbv.internal       : ok=0    changed=0    unreachable=0    failed=1
nas02.etsbv.internal       : ok=0    changed=0    unreachable=0    failed=1
```

So how do we fix this? Actually quite easy. However, we need to build
**sshpass** and install it from source.

So let's get that downloaded, built and installed.

```bash
cd ~/Downloads
wget https://downloads.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz
tar zxvf sshpass-1.06.tar.gz
cd sshpass-1.06
./configure
make
sudo make install
```

Now let's validate that **sshpass** is indeed installed now.

```raw
sshpass
Usage: sshpass [-f|-d|-p|-e] [-hV] command parameters
   -f filename   Take password to use from file
   -d number     Use number as file descriptor for getting password
   -p password   Provide password as argument (security unwise)
   -e            Password is passed as env-var "SSHPASS"
   With no parameters - password will be taken from stdin

   -P prompt     Which string should sshpass search for to detect a password prompt
   -v            Be verbose about what you're doing
   -h            Show help (this screen)
   -V            Print version information
At most one of -f, -d, -p or -e should be used
```

And sure enough, there it is now.

So now let's attempt to run our Ansible playbook once again.

**SUCCESS!!!!**

Now if you were using a **Linux** host such as **Ubuntu**, this would
have been as easy as:

```bash
sudo apt-get install sshpass
```

Interested in using Python virtual environments for Ansible regression
testing? Here is a script to help you with that.

{% gist mrlesmithjr/8beb3ab7989e5ee3ef61082c9162b564 %}

Enjoy!
