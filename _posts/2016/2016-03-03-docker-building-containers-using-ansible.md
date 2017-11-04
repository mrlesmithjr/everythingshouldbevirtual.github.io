---
  title: Docker - Building containers using Ansible
  categories:
    - Automation
    - Containers
  tags:
    - Ansible
    - Docker
  redirect_from:
    - /docker-building-containers-using-ansible
---

Have you thought about building Docker containers using some Ansible
role(s) or playbook(s) you may already be using for your regular
automated deployments? Well this is something I have been going back and
forth on. Whether to disregard existing Ansible roles  I have created to
deploy applications and services with or to start from scratch to build
Docker containers. So I wanted to share a quick example of how you can
do this if interested. The only downside is that the Docker image is a
little fat because of the Ansible installation within the container. The
plus to this is that Ansible is already installed in the container and
you can actually reconfigure your containers with different variables (
based on variables in your Ansible role(s) or playbook(s) ) using docker
exec commands. But that is debatable IMHO because I would possibly
rather use Ansible to spin up the containers passing the ENV variables
which are defined as part of the container. So the real benefit here
would be to leverage your existing role(s) and playbook(s) that you use
for your non containerized instances to build your Docker containers.

So with the above being said let's take these methodologies and build a
MySQL container.

As I mentioned above, I already have an Ansible role to deploy MySQL so
I will be leveraging that.

If we look at the directory structure of the files that will be copied
into our Docker container during the build.

```bash
vagrant@node-1:/vagrant/containers/mysql-ansible$ tree
.
├── ansible_tasks
│   ├── playbook.yml
│   └── requirements.yml
└── Dockerfile


1 directory, 3 files
```

And below are the contents of the files which will be copied.

playbook.yml
{% raw %}

```yaml
---
- hosts: localhost
  become: true
  vars:
    - mysql_allow_remote_connections: true
    - mysql_docker_install: true
  roles:
    - role: ansible-mysql
  tasks:
```

{% endraw %}
requirements.yml

```yaml
---
- src: https://github.com/mrlesmithjr/ansible-mysql.git
```

And our Dockerfile to build from (Using Ansible)

```bash
FROM ubuntu:14.04

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

#Update apt-cache
RUN apt-get update

#Install pre-reqs for Ansible
RUN apt-get -y install git python-dev python-pip

#Install Ansible
RUN pip install ansible

#Copy Ansible tasks
COPY ansible_tasks /opt/ansible_tasks

#Install Ansible role(s)
RUN ansible-galaxy install -r /opt/ansible_tasks/requirements.yml

#Run Ansible playbook
RUN ansible-playbook -c local /opt/ansible_tasks/playbook.yml

#Clean-up packages
RUN apt-get -y clean && \
    apt-get -y autoremove

#Clean-up temp files
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#Expose port(s)
EXPOSE 3306

#Process start-up
CMD ["/usr/bin/mysqld_safe"]
```

Now we can build our container..

```bash
docker build -t mysql-ansible .
```

Now if we do a standard Docker install without using Ansible using the
below Dockerfile

```bash
FROM ubuntu:14.04

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

#Update apt-cache
RUN apt-get update

#Install MySQL
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install mysql-server

#Clean-up packages
RUN apt-get -y clean && \
    apt-get -y autoremove

#Clean-up temp files
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#Expose port(s)
EXPOSE 3306

#Process start-up
CMD ["/usr/bin/mysqld_safe"]
```

And now build without Ansible.

```bash
docker build -t mysql .
```

And if we do a quick look at our Docker images to compare the
differences in the image sizes...

```bash
vagrant@node-1:/vagrant/containers/mysql$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              26c155e1472e        4 seconds ago       339.4 MB
mysql-ansible       latest              549207bd1f0e        14 minutes ago      636 MB
ubuntu              14.04               07c86167cdc4        3 hours ago         188 MB
```

As you can see there is quite a bit of bloat using Ansible but it may or
may not be worth it. So YMMV.

So there you have it and hopefully you found this to be of some use. And
I am very interested in your thoughts and perspecitves.

> UPDATE:
> After posting this I thought I would see if there was a difference in
> the image size if I were to install Ansible via apt ppa instead. Being
> that I installed Ansible via pip in the previous comparison. So here are
> those results. (The image names are different than the above original
> but you can compare the image sizes to validate)

Using the following Dockerfile...

```bash
FROM ubuntu:14.04

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

#Update apt-cache
RUN apt-get update

#Install pre-reqs for Ansible
RUN apt-get -y install git software-properties-common

#Adding Ansible ppa
RUN apt-add-repository ppa:ansible/ansible

#Update apt-cache
RUN apt-get update

#Install Ansible
RUN apt-get -y install ansible

#Copy Ansible tasks
COPY ansible_tasks /opt/ansible_tasks

#Install Ansible role(s)
RUN ansible-galaxy install -r /opt/ansible_tasks/requirements.yml

#Run Ansible playbook
RUN ansible-playbook -c local /opt/ansible_tasks/playbook.yml

#RUN pip uninstall ansible -y

#RUN apt-get -y purge git python-dev python-pip

#Clean-up packages
RUN apt-get -y clean && \
    apt-get -y autoremove

#Clean-up temp files
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#Expose port(s)
EXPOSE 3306

#Process start-up
CMD ["/usr/bin/mysqld_safe"]
```

And now building...

```bash
docker build -t mysql-ansible-apt .
```

And below are the results...

```bash
vagrant@node-1:/vagrant/containers$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql-ansible-pip   latest              b7b7f93506c6        9 seconds ago       636 MB
mysql-ansible-apt   latest              f9e313ffc5fa        2 minutes ago       504.9 MB
mysql               latest              9ac234dd93bf        4 minutes ago       339.4 MB
ubuntu              14.04               07c86167cdc4        3 hours ago         188 MB
```

And as you can see from above we indeed saved 132MB installing Ansible
via apt using their ppa repository.

Enjoy!
