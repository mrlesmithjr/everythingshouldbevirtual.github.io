---
  title: Ansible - Setting up an Ansible Control
---

In this post we will be setting up an Ansible Control Machine to execute
our Ansible tasks from. This server will not have writable access to our
Git repos so no changes can be made and pushed up. For our Ansible
Control Machine we will be using an Ubuntu 14.04 LTS server.

The focus of this was to setup a control machine in order to test and
push code based on a specific Ansible release. This was mainly due to
the release of 2.0 in which I have ran into several issues as far as
backwards compatibility and I do not want to go through and make changes
so soon on roles and such that have been working for quite some time
now. And also as I begin training others and learning the Infra-As-Code
journey.

If you are interested in testing this out in a Vagrant environment I
have put together a repo for going through these series of posts to
come. This Vagrant environment will spin up a [Gerrit Code Review](https://www.gerritcodereview.com/) server and a client in which
you can follow along with throughout this post. You can view the GitHub
repo [here](https://github.com/mrlesmithjr/vagrant-ansible-gerrit-lab).

> NOTE - This post will be updated over time so it may or not be
> complete at the time of reading.

To install python follow the below for your OS.

`Ubuntu`:

```bash
sudo apt-get update
sudo apt-get install python-setuptools python-dev libffi-dev libssl-dev git sshpass tree
sudo easy_install pip
```

`OSX`:
You will need to install [Xcode](https://developer.apple.com/xcode/)
developers kit first.
Next you will need to install [Homebrew](http://brew.sh/).

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Now you can install python.

```bash
sudo easy_install pip
```

Now with the above out of the way we are ready to begin.

```bash
sudo -H pip uninstall virtualenv
sudo -H pip uninstall virtualenvwrapper
sudo -H pip install virtualenv
sudo -H pip install virtualenvwrapper --ignore-installed six
sudo -H pip install httplib2
mkdir ~/.virtualenvs
```

If not using zsh and standard bash

```bash
echo "source "$(which virtualenvwrapper.sh) >> ~/.profile
echo "export WORKON_HOME=~/.virtualenvs" >> ~/.profile
source ~/.profile
```

If using zsh

```bash
echo "source "$(which virtualenvwrapper.sh) >> ~/.zshrc
echo "export WORKON_HOME=~/.virtualenvs" >> ~/.zshrc
source ~/.zshrc
```

Now let's create our Python virtual environments for testing different
versions of Ansible.

```bash
mkdir ~/ansible_virtualenvs

cd ~/ansible_virtualenvs
mkdir 1.9.4
cd 1.9.4
mkvirtualenv ansible-1.9.4
pip install ansible==1.9.4
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 1.9.5
cd 1.9.5
mkvirtualenv ansible-1.9.5
pip install ansible==1.9.5
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 1.9.6
cd 1.9.6
mkvirtualenv ansible-1.9.6
pip install ansible==1.9.6
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.0.1
cd 2.0.0.1
mkvirtualenv ansible-2.0.0.1
pip install ansible==2.0.0.1
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.0.2
cd 2.0.0.2
mkvirtualenv ansible-2.0.0.2
pip install ansible==2.0.0.2
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.1.0
cd 2.0.1.0
mkvirtualenv ansible-2.0.1.0
pip install ansible==2.0.1.0
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.2.0
cd 2.0.2.0
mkvirtualenv ansible-2.0.2.0
pip install ansible==2.0.2.0
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.1.0.0
cd 2.1.0.0
mkvirtualenv ansible-2.1.0.0
pip install ansible==2.1.0.0
ansible --version
deactivate
```

Now we are ready to start working within our different Ansible version
environments.

So let's first setup our 1.9.4 environment.

```bash
workon ansible-1.9.4
pip install pyopenssl ndg-httpsclient pyasn1 pysphere pyvmomi git-review
```

Now we will setup our 2.0 environment.

```bash
deactivate

workon ansible-2.0
pip install pyopenssl ndg-httpsclient pyasn1 pysphere pyvmomi git-review
```

Now let's create our Git Projects directory to store all of our Git
Projects and code.

```bash
mkdir ~/Git_Projects
cd ~/Git_Projects
```

Now we are ready to start populating our GitHub repo(s) with the code we
want to use and/or just view.

I have a GitHub repo that has some Ansible tasks that will allow us to
pull down our specific required repos.

```bash
cd ~/Git_Projects
git clone https://github.com/mrlesmithjr/ansible-clone-git-repos.git
```

Once the code is pulled down you can modify the defaults/main.yml file
to comment out or add additional GitHub user(s) repos to pull down.

```bash
cd ansible-clone-git-repos
vi defaults/main.yml
```

You will see the default is the following:

```yaml
github_users:  #define github user(s) to clone repos from
  - bunchc
  - debops
  - lowescott
  - Mierdin
  - mrlesmithjr
  - dstamen
  - phpipam
```

I will just be pulling down my own personal repos so I will comment out
the other as such:

```yaml
github_users:  #define github user(s) to clone repos from
#  - bunchc
#  - debops
#  - lowescott
#  - Mierdin
  - mrlesmithjr
#  - dstamen
#  - phpipam
```

Now save the file.

Let's now use our Ansible virtual environment of 1.9.4 here on out. (
We can easily switch between Ansible versions in order to conduct
testing from our same control machine. )

```bash
workon ansible-1.9.4
./clone_git_repos.sh
```

Once all of the repos have been pulled down you can take a look at them.

```bash
cd ../GitHub/mrlesmithjr
ls
```

And below is a script you can run to perform all of the steps above if
you want to take the easy way :)

```bash
#!/bin/bash
sudo apt-get update
sudo apt-get -y install python-setuptools python-dev libffi-dev libssl-dev git sshpass
sudo easy_install pip

sudo -H pip uninstall -y virtualenv
sudo -H pip uninstall -y virtualenvwrapper
sudo -H pip install virtualenv
sudo -H pip install virtualenvwrapper --ignore-installed six
sudo -H pip install httplib2

rm -rf ~/.virtualenvs
mkdir ~/.virtualenvs
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.profile
echo "export WORKON_HOME=~/.virtualenvs" >> ~/.profile
source ~/.profile

rm -rf ~/ansible_virtualenvs
mkdir ~/ansible_virtualenvs

cd ~/ansible_virtualenvs
mkdir 1.9.4
cd 1.9.4
mkvirtualenv ansible-1.9.4
pip install ansible==1.9.4
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 1.9.5
cd 1.9.5
mkvirtualenv ansible-1.9.5
pip install ansible==1.9.5
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 1.9.6
cd 1.9.6
mkvirtualenv ansible-1.9.6
pip install ansible==1.9.6
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.0.1
cd 2.0.0.1
mkvirtualenv ansible-2.0.0.1
pip install ansible==2.0.0.1
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.0.2
cd 2.0.0.2
mkvirtualenv ansible-2.0.0.2
pip install ansible==2.0.0.2
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.1.0
cd 2.0.1.0
mkvirtualenv ansible-2.0.1.0
pip install ansible==2.0.1.0
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.0.2.0
cd 2.0.2.0
mkvirtualenv ansible-2.0.2.0
pip install ansible==2.0.2.0
ansible --version
deactivate

cd ~/ansible_virtualenvs
mkdir 2.1.0.0
cd 2.1.0.0
mkvirtualenv ansible-2.1.0.0
pip install ansible==2.1.0.0
ansible --version
deactivate

mkdir ~/Git_Projects
cd ~/Git_Projects
git clone https://github.com/mrlesmithjr/ansible-clone-git-repos.git

cd ansible-clone-git-repos
sed -i -e 's|- bunchc|#- bunchc|' defaults/main.yml
sed -i -e 's|- debops|#- debops|' defaults/main.yml
sed -i -e 's|- lowescott|#- lowescott|' defaults/main.yml
sed -i -e 's|- Mierdin|#- Mierdin|' defaults/main.yml
sed -i -e 's|- dstamen|#- dstamen|' defaults/main.yml
sed -i -e 's|- phpipam|#- phpipam|' defaults/main.yml

workon ansible-1.9.4
./clone_git_repos.sh
cd ../GitHub/mrlesmithjr
ls
```

Now that we have some roles downloaded we can begin using them but we
first need to decide how we will leverage using these roles. There are
several ways that we can do this of course. But now is the time to make
that decision.

By default even in our Ansible virtual environment our roles would be
installed and looked for in the normal directory of
**_/etc/ansible/roles_**.

There are reasons to store all of your roles there and also there are
reasons not to. I am only providing what I find to be the best for my
environment(s). I personally choose to store all of my roles within a
folder called roles which would be part of my actual project. If you are
looking for a way to consolidate roles and ensure that all projects use
the same common roles that works very well too. I do this by collecting
all of my roles into an internal Git repo called
**ansible-common-core-roles**. And when I create a new project I pull
these roles from my internal Git repo as below:

```bash
git clone git@gitserver:/ansible-common-core-roles.git roles
```

Now I can leverage the same roles throughout each project as well as
others can too. I also do this for common variables and inventory to be
used over and over again for consistency. I use a repo called
**ansible-environments** and when I need those I will pull them down as
below:

```bash
git clone git@gitserver:/ansible-environments.git environments
```

Now when I need to run a play I can leverage all of these common roles
and variables as below:

```bash
ansible-playbook -i environments/production/site/inventory playbook.yml
```

As you can from above I reference production/site which is where we can
store site specific group_vars and host_vars. This allows for
variables to be site specific.

Another method that I use is when I might want to create a semi-private
repository to ensure that I or someone else has not updated the
ansible-common-core-roles. Or I need a role that is specific to that
project. Keep in mind I may still want to leverage upstream Git repos
for my roles but I want to ensure they are only updated when I choose.
For this method I will show an example of how to set this up.

First we need to create our directory structure for our LAMP Stack
project.

```bash
cd ~/Git_Projects
mkdir -p Lab/LAMP
cd Lab/LAMP
mkdir environments roles
git init
```

Now for this project in order to define where my roles are located I
will create an **ansible.cfg** file to use. Keep in mind by default when
running an Ansible playbook it will also look in the current directory
for a roles directory. I am doing this to ensure that I am using this
specific roles directory and not the default **/etc/ansible/roles**
directory. This will ensure that if I or someone else placed a role in
that directory it will not accidentally be used. Just a precaution.

So let's create our ansible.cfg file in our project folder. You can
check out the additional options you can use in your **ansible.cfg**
[here](http://docs.ansible.com/ansible/intro_configuration.html). Keep
in mind this will only apply any differences in the default
**ansible.cfg** which is installed as part of the Ansible package.

```bash
vi ansible.cfg
```

And we will add the following to this file.

```bash
[defaults]
roles_path = ./roles
```

We can also add multiple roles_path locations if we desire that as well
by doing the following:

```bash
roles_path = ./roles:/opt/ansible/roles
```

But we will only defining one path for this project.

Now we are ready to being creating our project. So first we know that
for a LAMP stack we will need Apache, MySQL and PHP/Python/Perl. Well in
this example I know that from the roles we pulled down from my GitHub
repo have those roles so I will leverage those for this project.

So first thing I will do is create a **requirements.yml** file which is
where we will define which roles from different resources I might want
to leverage.

```bash
vi requirements.yml
```

And we will add the following bit of information to this file.

```yaml
---
- src: https://github.com/mrlesmithjr/ansible-apache2.git
- src: https://github.com/mrlesmithjr/ansible-mysql.git
```

The above states that we will be pulling the repos defined. And in order
to pull these roles down we will run the following:

```bash
ansible-galaxy install -r requirements.yml
```

Once that has completed we can ensure that we did indeed pull down the
roles we defined and they were installed to our roles directory as
defined in our **ansible.cfg** file.

```bash
cd roles
ls

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP/roles$ ls
ansible-apache2  ansible-mysql
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP/roles$
```

So we did indeed install them as required and they were placed in the
correct folder. Now say we want to ensure we have the most current roles from our
upstream Git repo and we executed the same command as we did previously:

```bash
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ansible-galaxy install -r requirements.yml
- executing: git clone https://github.com/mrlesmithjr/ansible-apache2.git ansible-apache2
- executing: git archive --prefix=ansible-apache2/ --output=/tmp/tmp8Xzo8j.tar HEAD
- extracting ansible-apache2 to ./roles/ansible-apache2
- error: the specified role ansible-apache2 appears to already exist. Use --force to replace it.
- ansible-apache2 was NOT installed successfully.
- you can use --ignore-errors to skip failed roles.
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

As you can see we received errors stating that the roles already exist.
So we can get around this if we force them to be installed again by
doing the following:

```bash
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ansible-galaxy install -r requirements.yml -f
- executing: git clone https://github.com/mrlesmithjr/ansible-apache2.git ansible-apache2
- executing: git archive --prefix=ansible-apache2/ --output=/tmp/tmp6JgwxN.tar HEAD
- extracting ansible-apache2 to ./roles/ansible-apache2
- ansible-apache2 was installed successfully
- executing: git clone https://github.com/mrlesmithjr/ansible-mysql.git ansible-mysql
- executing: git archive --prefix=ansible-mysql/ --output=/tmp/tmpkc5W2l.tar HEAD
- extracting ansible-mysql to ./roles/ansible-mysql
- ansible-mysql was installed successfully
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

Now we have success and we now have the latest roles from our upstream
Git repo using our **requirements.yml** file. You could also easily just
pull down roles without using a **requirements.yml** file by doing the
following:

```bash
cd roles
git clone https://github.com/mrlesmithjr/ansible-apache2.git
git clone https://github.com/mrlesmithjr/ansible-mysql.git
```

Now in order to pull the latest code you can change into the roles/role
directory and do the following:
We will be checking the ansible-apache2 role.

```bash
cd roles/ansible-apache2
git pull

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP/roles/ansible-apache2$ git pull
Already up-to-date.
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP/roles/ansible-apache2$
```

As you can see the role is already up-to-date. One reason not to do this
(as far as my experience has been) is that because this is part of
another Git repo, if we store our project on an internal Git repo, when
we push our code up it will not include any directories that are pulled
from another upstream repo. So I generally will just use a
**requirements.yml** file and update the roles this way. Remember before
doing that you may want to create a new Git branch for testing first.

```bash
git checkout -b test

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ git checkout -b test
Switched to a new branch 'test'
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$

ansible-galaxy install -r requirements.yml -f

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ansible-galaxy install -r requirements.yml -f
- executing: git clone https://github.com/mrlesmithjr/ansible-apache2.git ansible-apache2
- executing: git archive --prefix=ansible-apache2/ --output=/tmp/tmp2HWi5u.tar HEAD
- extracting ansible-apache2 to ./roles/ansible-apache2
- ansible-apache2 was installed successfully
- executing: git clone https://github.com/mrlesmithjr/ansible-mysql.git ansible-mysql
- executing: git archive --prefix=ansible-mysql/ --output=/tmp/tmp55rV1S.tar HEAD
- extracting ansible-mysql to ./roles/ansible-mysql
- ansible-mysql was installed successfully
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

Now this is not much use for our internal Git repo at this time because
we have not set that up yet. I will leave this up to you to decide on
your internal Git repo server and setup. I choose to use Gerrit because
I want code review abilities. If you are interested in setting up a
Gerrit server I have an **ansible-gerrit** role that can be leveraged
for that as well. You can view this in the
~/**Git_Projects/GitHub/mrlesmithjr/ansible-gerrit** directory that we
pulled down in the beginning of this post. _( I will be publishing
another post on setting up a Gerrit server later and will reference it
from here if interested. )_

```bash
cd ~/Git_Projects/GitHub/mrlesmithjr/ansible-gerrit/
```

Now let's create an Ansible playbook to leverage our roles for our LAMP
project.

Before we begin let's ensure that we are in our LAMP project directory.

```bash
cd ~/Git_Projects/Lab/LAMP/
```

Now we will create a playbook.yml file as below:

```bash
vi playbook.yml
```

```yaml
---
- name: Build LAMP Servers
  hosts: lamp-servers
  become: true
  vars:
    - install_php: true
  roles:
    - role: ansible-apache2
    - role: ansible-mysql
  tasks:
```

Now with our playbook created we need to create our inventory file and
our group_vars and host_vars.

```bash
cd environments
mkdir -p dev/group_vars/all dev/host_vars prod/group_vars/all prod/host_vars test/group_vars/all test/host_vars
```

To see all of our environment folders we just created:

```bash
tree

(ansible-1.9.4)vagrant@client:~/Git_Projects/Lab/LAMP/environments$ tree
.
├── dev
│   ├── group_vars
│   │   └── all
│   └── host_vars
├── prod
│   ├── group_vars
│   │   └── all
│   └── host_vars
└── test
    ├── group_vars
    │   └── all
    └── host_vars

12 directories, 0 files
(ansible-1.9.4)vagrant@client:~/Git_Projects/Lab/LAMP/environments$
```

We will work within our dev environment here.

```bash
cd environments/dev
```

Now create your inventory file.

```bash
vi inventory
```

And add the following:

```bash
[lamp-servers]
lamp-01
lamp-02
```

Now if you look at our playbook we defined a variable there which is
install_php: true we could also not define this variable within our
playbook but within our group_vars for our dev environment.

```bash
cd group_vars
mkdir lamp-servers
cd lamp-servers
vi apache.yml
```

Now paste the following:

```yaml
---
install_php: true
```

Now we can remove that variable definition from our playbook as our
group_vars will now cover that variable.

```bash
cd ../../../../
vi playbook.yml
```

Now remove the line with the following:

```yaml
- install_php: true
```

So our playbook should now look like below:

```yaml
---
- name: Build LAMP Servers
  hosts: lamp-servers
  become: true
  vars:
  roles:
    - role: ansible-apache2
    - role: ansible-mysql
  tasks:
```

Now we should have everything in place except our servers to provision
which I will not be going into here.

So now to kick off the Ansible playbook to test out we would run the
following:

```bash
ansible-playbook -i environments/dev/inventory playbook.yml --user remote --ask-pass --ask-sudo-pass
```

If you notice above we are specifying the user and prompting for
passwords. We could get around this requirement and add our ssh public
key to the remote servers we will be managing (Highly recommend). You
could follow your normal process of accomplishing this or you can check
out the Ansible role called ansible-manage-ssh-keys that we pulled down
in the beginning of this post.

```bash
cd ~/Git_Projects/GitHub/mrlesmithjr/ansible-manage-ssh-keys/
```

But for this exercise let's go ahead and add that role to our
requirements.yml file and pull that down to our roles directory as well.

```bash
vi requirements.yml
```

And paste the below at the end of the file:

```yaml
- src: https://github.com/mrlesmithjr/ansible-manage-ssh-keys.git
```

So our requirements.yml should now look like below:

```yaml
---
- src: https://github.com/mrlesmithjr/ansible-apache2.git
- src: https://github.com/mrlesmithjr/ansible-mysql.git
- src: https://github.com/mrlesmithjr/ansible-manage-ssh-keys.git
```

Now let's pull down our requirements once again. Remember we will get
errors if the role already exists so you may want to force the install
as previously shown or if you want to ensure that you do not pull any
changes in our upstream repo do not force the install.

```bash
ansible-galaxy install -r requirements.yml

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ansible-galaxy install -r requirements.yml
- executing: git clone https://github.com/mrlesmithjr/ansible-apache2.git ansible-apache2
- executing: git archive --prefix=ansible-apache2/ --output=/tmp/tmp53JbAU.tar HEAD
- extracting ansible-apache2 to ./roles/ansible-apache2
- error: the specified role ansible-apache2 appears to already exist. Use --force to replace it.
- ansible-apache2 was NOT installed successfully.
- you can use --ignore-errors to skip failed roles.
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

As before we received errors and if you check our roles directory the
new role was not pulled down either.

```bash
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ls roles
ansible-apache2  ansible-mysql
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

How can we get around this without forcing the install? If you notice in
the messages above there is a reference to using --ignore-errors. We
can leverage that parameter in order to pull down our additional roles
without touching the previously existing roles.

```bash
ansible-galaxy install -r requirements.yml -i

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ansible-galaxy install -r requirements.yml -i
- executing: git clone https://github.com/mrlesmithjr/ansible-apache2.git ansible-apache2
- executing: git archive --prefix=ansible-apache2/ --output=/tmp/tmpNuZPUo.tar HEAD
- extracting ansible-apache2 to ./roles/ansible-apache2
- error: the specified role ansible-apache2 appears to already exist. Use --force to replace it.
- ansible-apache2 was NOT installed successfully.
- executing: git clone https://github.com/mrlesmithjr/ansible-mysql.git ansible-mysql
- executing: git archive --prefix=ansible-mysql/ --output=/tmp/tmpLQFR1m.tar HEAD
- extracting ansible-mysql to ./roles/ansible-mysql
- error: the specified role ansible-mysql appears to already exist. Use --force to replace it.
- ansible-mysql was NOT installed successfully.
- executing: git clone https://github.com/mrlesmithjr/ansible-manage-ssh-keys.git ansible-manage-ssh-keys
- executing: git archive --prefix=ansible-manage-ssh-keys/ --output=/tmp/tmp2RVBbl.tar HEAD
- extracting ansible-manage-ssh-keys to ./roles/ansible-manage-ssh-keys
- ansible-manage-ssh-keys was installed successfully
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

Now you see that at the end our ansible-manage-ssh-keys role was
successfully installed. And to verify:

```bash
ls roles/

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ls roles/
ansible-apache2  ansible-manage-ssh-keys  ansible-mysql
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

Now we see our additional role as being installed.

Now we are ready to setup our ssh keys to initiate ssh password-less
sessions to run our Ansible plays against our servers.

```raw
ssh-keygen

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/remote/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/remote/.ssh/id_rsa.
Your public key has been saved in /home/remote/.ssh/id_rsa.pub.
The key fingerprint is:
98:bc:e8:9c:a8:a3:c2:54:eb:2b:b9:9d:33:a1:cb:78 remote@ansible-control
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|                 |
|   . . o         |
|  . . + S        |
| . o . .         |
|o + o .          |
|==EB..           |
|*B=+B            |
+-----------------+
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

Now that we have generated our users ssh keys (remote is username in
this case) we are now ready to setup our ability to deploy these keys.

So let's create the directory in which this role will look for our ssh
keys to deploy.

```bash
mkdir ssh_pub_keys
```

Now we need to copy our ssh public key that we just generated and we
will give it a name of the username@hostname.pub to easily identify.

```bash
cp ~/.ssh/id_rsa.pub ssh_pub_keys/$USER@$HOSTNAME.pub
```

And if we take a look at our ssh_pub_keys directory:

```bash
ls ssh_pub_keys/

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ls ssh_pub_keys/
remote@ansible-control.pub
(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$
```

Now before we go any further we need to decide whether we will be using
an account that already exists on our remote servers to add our ssh pub
to or if we will be creating a new account for remote provisioning of
our Ansible roles. I prefer to use an account called remote but this can
be any account that you desire which has sudo access and preferably no
password for sudo. **_(DO NOT USE ROOT). _**If you need to setup user
accounts you can leverage the Ansible role called ansible-users that we
pulled down in the beginning of this post. And have a look at that and
leverage the role if desired. For this example I will be leveraging this
role to keep consistency.

So I will add the following additional role to our requirements.yml file
and then install the additional role.

```bash
cd ~/Git_Projects/Lab/LAMP
vi requirements.yml
```

Add the following additional role:

```yaml
- src: https://github.com/mrlesmithjr/ansible-users.git
```

Now our requirements.yml should look like below:

```yaml
---
- src: https://github.com/mrlesmithjr/ansible-apache2.git
- src: https://github.com/mrlesmithjr/ansible-mysql.git
- src: https://github.com/mrlesmithjr/ansible-manage-ssh-keys.git
- src: https://github.com/mrlesmithjr/ansible-users.git
```

And again let's install our requirements. (Remember we will be ignoring
errors (-i) but you can force (-f) the install as well)

```bash
ansible-galaxy install -r requirements.yml -i

(ansible-1.9.4)remote@ansible-control:~/Git_Projects/Lab/LAMP$ ansible-galaxy install -r requirements.yml -i
- executing: git clone https://github.com/mrlesmithjr/ansible-apache2.git ansible-apache2
- executing: git archive --prefix=ansible-apache2/ --output=/tmp/tmp2xUow3.tar HEAD
- extracting ansible-apache2 to ./roles/ansible-apache2
- error: the specified role ansible-apache2 appears to already exist. Use --force to replace it.
- ansible-apache2 was NOT installed successfully.
- executing: git clone https://github.com/mrlesmithjr/ansible-mysql.git ansible-mysql
- executing: git archive --prefix=ansible-mysql/ --output=/tmp/tmpyT61RC.tar HEAD
- extracting ansible-mysql to ./roles/ansible-mysql
- error: the specified role ansible-mysql appears to already exist. Use --force to replace it.
- ansible-mysql was NOT installed successfully.
- executing: git clone https://github.com/mrlesmithjr/ansible-manage-ssh-keys.git ansible-manage-ssh-keys
- executing: git archive --prefix=ansible-manage-ssh-keys/ --output=/tmp/tmpCOJ05g.tar HEAD
- extracting ansible-manage-ssh-keys to ./roles/ansible-manage-ssh-keys
- error: the specified role ansible-manage-ssh-keys appears to already exist. Use --force to replace it.
- ansible-manage-ssh-keys was NOT installed successfully.
- executing: git clone https://github.com/mrlesmithjr/ansible-users.git ansible-users
- executing: git archive --prefix=ansible-users/ --output=/tmp/tmpIy507F.tar HEAD
- extracting ansible-users to ./roles/ansible-users
- ansible-users was installed successfully
```

Now that the ansible-users role has been installed we need to take a
look and see what we need to add to create a new user called remote on
our servers which we will attach our ssh keys to and use for Ansible
deployments.
So if you look at `roles/ansible-users/defaults/main.yml`

```yaml
---
# defaults file for ansible-users
create_local_users: true  #defines creating local user accounts on hosts
create_users:  #defines user accounts to setup on hosts....define here or in group_vars/all
  - user: demo_user  #define username
    authorized_keys: ''
    comment: 'Demo user'  #define a comment to associate with the account
    generate_keys: false  #generate ssh keys...true|false
    home: ''  #define a different home directory... ''=/home/username
    pass: demo_password  #define password for account
    setup: false  #true=creates account|false=removes account if exists...true|false
    shell: ''  #define a different shell for the user
    preseed_user: false  #defines if user should be setup as default user during preseed auto-install...Only 1 user can be added...used in tftpserver Ansible role (mrlesmithjr.tftpserver or ansible-tftpserver)
    sudo: false  #define if user should have sudo access...true|false
    system_account: false  #define if account is a system account...true|falseinstall_fail2ban: false
```

From the above we will need to define our user(s) to create on our
remote servers. In my case I will be creating a user named remote and
granting sudo access to this account. Now we could easily add our
account information to the above file but rather than doing that we will
create a new group_vars file that will contain our accounts. We can
either define this for the lamp-servers group or the all group. Defining
on lamp-servers will only apply to our lamp-servers whereas if we define
in all, the account(s) will be created on all of our servers and not
just lamp-servers.

So let's create our group_vars/all variable file for our Dev
environment.

```bash
mkdir environments/dev/group_vars/all
vi environments/dev/group_vars/all/accounts.yml
```

And add the following..Modify of course for your environment.

```yaml
---
create_users:  #defines user accounts to setup on hosts....define here or in group_vars/all
  - user: remote  #define username
    authorized_keys: ''
    comment: 'Ansible remote user'  #define a comment to associate with the account
    generate_keys: false  #generate ssh keys...true|false
    home: ''  #define a different home directory... ''=/home/username
    pass: password1  #define password for account
    setup: true  #true=creates account|false=removes account if exists...true|false
    shell: ''  #define a different shell for the user
    preseed_user: false  #defines if user should be setup as default user during preseed auto-install...Only 1 user can be added...used in tftpserver Ansible role (mrlesmithjr.tftpserver or ansible-tftpserver)
    sudo: true  #define if user should have sudo access...true|false
    system_account: false  #define if account is a system account...true|falseinstall_fail2ban: false
```

Now that is set we also need to define our ssh pub key information for
our ansible-manage-ssh-keys role. So if you look at the default vars for
this role.

```yaml
---
# defaults file for ansible-manage-ssh-keys
enable_manage_ssh_keys: false  #defines if remote ssh keys should be managed
manage_ssh_keys:
  - remote_user: demo_user  #define username on remote system to add defined keys to
    present: true  #defines if ssh key should be added or removed
    keys:  #define key(s) to add to remote username
      - ssh_pub_keys/demo_user.pub
      - ssh_pub_keys/demo_user_1.pub
  - remote_user: demo_user2
    present: false
    keys:
      - ssh_pub_keys/demo_user2.pub
```

From the above default variables we need to add the info for our
generated keys and associate that with the account (remote) that we will
be creating on our servers. So let's do that by adding this information
to the same file we created above to add our user(s) to. This way we can
keep everything together in an accounts related file and apply security
against it if desired (recommended) using ansible-vault (More on this
another time).

```bash
vi environments/dev/group_vars/all/accounts.yml
```

And modify the above variables to match the below and append them to our
accounts.yml file.

```yaml
enable_manage_ssh_keys: true  #defines if remote ssh keys should be managed
manage_ssh_keys:
  - remote_user: remote  #define username on remote system to add defined keys to
    present: true  #defines if ssh key should be added or removed
    keys:  #define key(s) to add to remote username
      - ssh_pub_keys/remote@ansible-control.pub
```

Now our accounts.yml file should like below:

```yaml
---
create_users:  #defines user accounts to setup on hosts....define here or in group_vars/all
  - user: remote  #define username
    authorized_keys: ''
    comment: 'Ansible remote user'  #define a comment to associate with the account
    generate_keys: false  #generate ssh keys...true|false
    home: ''  #define a different home directory... ''=/home/username
    pass: password1  #define password for account
    setup: true  #true=creates account|false=removes account if exists...true|false
    shell: ''  #define a different shell for the user
    preseed_user: false  #defines if user should be setup as default user during preseed auto-install...Only 1 user can be added...used in tftpserver Ansible role (mrlesmithjr.tftpserver or ansible-tftpserver)
    sudo: true  #define if user should have sudo access...true|false
    system_account: false  #define if account is a system account...true|falseinstall_fail2ban: false
enable_manage_ssh_keys: true  #defines if remote ssh keys should be managed
manage_ssh_keys:
  - remote_user: remote  #define username on remote system to add defined keys to
    present: true  #defines if ssh key should be added or removed
    keys:  #define key(s) to add to remote username
      - ssh_pub_keys/remote@ansible-control.pub
```

Now that we have all of the above taken care of we need to make sure our
playbook has all of the appropriate roles added. And it should now look
like below:

```yaml
---
- name: Build LAMP Servers
  hosts: lamp-servers
  become: true
  vars:
  roles:
    - role: ansible-users
    - role: ansible-manage-ssh-keys
    - role: ansible-apache2
    - role: ansible-mysql
  tasks:
```

## Tweaks

Now how about some tweaking to speed things up? Well one thing we can do
is enable fact caching. Ansible by default allows you to either use
Redis or a jsonfile to cache facts that are gathered. I prefer to use
Redis but that is just my preference. Depending on your choice you can
follow the respective section below.

`Redis`:
We need to first install redis and we can do so as below:

```bash
sudo apt-get install redis-server
```

If you are using OSX then you can install redis as follows using
[homebrew](http://brew.sh/):

```bash
brew install redis
ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
```

Now you can validate that redis is running:
`Ubuntu/Debian`:

```bash
sudo service redis-server status
```

`OSX`:

```bash
brew services list | grep redis
```

Now let's configure Ansible to leverage redis and we will do that by
editing our ansible.cfg (You can do this globally or per project. We
will be doing it at a project level).

```bash
cd ~/Git_Projects/Lab/LAMP
vi ansible.cfg
```

And add the following to the end of the file.

```bash
gathering = smart
fact_caching = redis
fact_caching_timeout = 86400
```

And our ansible.cfg file should look like the following now:

```bash
[defaults]
roles_path = ./roles
gathering = smart
fact_caching = redis
fact_caching_timeout = 86400
```

Now we need to install the redis python module in each of our Python
Virtual Environments.

`ansible-1.9.4`

```bash
deactivate
workon ansible-1.9.4
pip install redis
```

Now let's configure our 2.0 environment.

```bash
deactivate
workon ansible-2.0
pip install redis
```

Now we can now leverage Redis for our fact_caching which will speed up
our plays when they start.

## Updating Python Modules

How can we update all of our Python modules for each of our Virtual
Environments? Actually quite easy. Just do the following in each
environment:

```bash
pip freeze | xargs pip install -U
```

So we would do the following for our two Ansible Virtual Environments.\
`ansible-1.9.4`

```bash
deactivate
workon ansible-1.9.4
pip freeze | xargs pip install -U
```

`ansible-2.0`

```bash
deactivate
workon ansible-2.0
pip freeze | xargs pip install -U
```

Now all of our Python modules for each Virtual Environment are
up-to-date.

## Additional Ansible Tools

Below is a list of some additional tools that you may want to checkout
as well.

[Ansible-Toolbox](https://github.com/dellis23/ansible-toolkit)

```bash
pip install ansible-toolbox
```

And there you have it. This will conclude Part-1 of setting up an
Ansible Control Machine. Up next will be actually beginning our
preparation of our remote servers leveraging these roles that we just
finished preparing.

Feel free to leave feedback as I am interested in hearing from others
and hopefully this has been useful.

Enjoy!
