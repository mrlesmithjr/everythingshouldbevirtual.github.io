---
  title: Docker - Ansible - ELK Stack
  categories:
    - Automation
    - Containers
    - Monitoring
  tags:
    - Ansible
    - Docker
    - ELK
---

In this post we will be going over setting up a quick and easy way to
standup ELK Stack using Docker containers for each of our components
required. Keep in mind that this will just cover the basics and not
going into depth on advanced GROK filtering and etc. But you can
definitely add that functionality if the desire is there. Now I have
chosen a different method of creating and maintaining my Docker
containers by using Ansible to do most of the configurations vs. using a
ton of bash. The downside is that the containers are a little larger is
size due to the overhead of Ansible being installed. I can deal with
that though for the flexibility. Also I am keeping an eye the
[ansible-container](https://www.ansible.com/ansible-container) solution
as well as it seems it is more along the lines of how I choose to
maintain my containers. I also published another
[post](https://everythingshouldbevirtual.com/docker-building-containers-using-ansible)
earlier this year around the idea of using Ansible to provision
containers but was actually leveraging some of the Ansible roles that I
had already created. I have chosen to not do this going forward (at
least for now).

I would like to touch briefly on each component providing the
**_Dockerfile_** and **_playbook.yml_** used. For each component I have
also included the link to the corresponding GitHub repository for
reference and more detail.

[**Elasticsearch**](https://github.com/mrlesmithjr/docker-ansible-elasticsearch)

`Dockerfile`:

```bash
FROM ubuntu:14.04

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

# Update apt-cache
RUN apt-get update

# Install Ansible
RUN apt-get -y install git software-properties-common && \
    apt-add-repository ppa:ansible/ansible && \
    apt-get update && \
    apt-get -y install ansible

# Copy Ansible Playbook
COPY playbook.yml /playbook.yml

# Run Ansible playbook
RUN ansible-playbook -i "localhost," -c local /playbook.yml

# Cleanup
RUN apt-get -y clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PATH /usr/share/elasticsearch/bin:$PATH

WORKDIR /usr/share/elasticsearch

# Setup entrypoint Ansible Playbook
COPY docker-entrypoint.yml /docker-entrypoint.yml

# Setup entrypoint script
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]

COPY config /usr/share/elasticsearch/config

# Setup volume
VOLUME /usr/share/elasticsearch/data

# Expose port(s)
EXPOSE 9200 9300

# Container start-up
CMD ["elasticsearch"]
```

`playbook.yml`:

{% raw %}

```yaml
---
- hosts: localhost
  connection: local
  become: true
  vars:
    - gosu_ver: 1.9
  tasks:
    - name: Installing apt-transport-https
      apt:
        name: "apt-transport-https"
        state: "present"

    - name: Installing ca-certificates
      apt:
        name: "ca-certificates"
        state: "latest"

    - name: Installing dumb-init
      apt:
        deb: "https://github.com/Yelp/dumb-init/releases/download/v1.0.2/dumb-init_1.0.2_amd64.deb"

    - name: Installing gosu
      get_url:
        url: "https://github.com/tianon/gosu/releases/download/{{ gosu_ver }}/gosu-amd64"
        dest: "/usr/local/bin/gosu"
        mode: 0755

    - name: Installing JRE
      apt:
        name: "default-jre-headless"
        state: "present"

    - name: Adding Elasticsearch Repo Key
      apt_key:
        url: "https://packages.elastic.co/GPG-KEY-elasticsearch"
        state: "present"

    - name: Adding Elasticsearch Repo
      apt_repository:
        repo: "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main"

    - name: Installing Elasticsearch
      apt:
        name: "elasticsearch"
        state: "present"
        install_recommends: no

    - name: Ensuring Elasticsearch Folders Exist
      file:
        path: "/usr/share/elasticsearch/{{ item }}"
        state: "directory"
        owner: "elasticsearch"
        group: "elasticsearch"
      with_items:
        - 'data'
        - 'logs'
        - 'config'
        - 'config/scripts'
```

{% endraw %}
[**Logstash**](https://github.com/mrlesmithjr/docker-ansible-logstash)

`Dockerfile`:

```bash
FROM ubuntu:14.04

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

# Update apt-cache
RUN apt-get update

# Install Ansible
RUN apt-get -y install git software-properties-common && \
    apt-add-repository ppa:ansible/ansible && \
    apt-get update && \
    apt-get -y install ansible

# Copy Ansible Playbook
COPY playbook.yml /playbook.yml

# Run Ansible playbook
RUN ansible-playbook -i "localhost," -c local /playbook.yml

# Cleanup
RUN apt-get -y clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PATH /opt/logstash/bin:$PATH

# necessary for 5.0+ (overriden via "--path.settings", ignored by < 5.0)
ENV LS_SETTINGS_DIR /etc/logstash
# comment out some troublesome configuration parameters
#   path.log: logs should go to stdout
#   path.config: No config files found: /etc/logstash/conf.d/*
RUN set -ex \
    && if [ -f "$LS_SETTINGS_DIR/logstash.yml" ]; then \
        sed -ri 's!^(path.log|path.config):!#&!g' "$LS_SETTINGS_DIR/logstash.yml"; \
    fi

# Setup entrypoint Ansible Playbook
COPY docker-entrypoint.yml /docker-entrypoint.yml

# Setup entrypoint script
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]

COPY config/ /etc/logstash/conf.d

# Setup volume
VOLUME /etc/logstash/conf.d

# Expose Port(s)
EXPOSE 514 514/udp 5044 10514 10514/udp

# Container start-up
CMD ["logstash", "agent", "-f", "/etc/logstash/conf.d/"]
```

`playbook.yml`:
{% raw %}

```yaml
---
- hosts: localhost
  connection: local
  become: true
  vars:
    - gosu_ver: '1.9'
    - logstash_ver: '2.3'
  tasks:
    - name: Installing apt-transport-https
      apt:
        name: "apt-transport-https"
        state: "present"

    - name: Installing ca-certificates
      apt:
        name: "ca-certificates"
        state: "latest"

    - name: Installing Logstash Plugins Pre-Reqs
      apt:
        name: "{{ item }}"
        state: "present"
        install_recommends: no
      with_items:
        - 'libzmq3'

    - name: Ensuring /usr/local/lib Exists
      file:
        path: "/usr/local/lib"
        state: "directory"

    # - name: Symlinking libzmq.so
    #   file:
    #     src: "/usr/lib/*/libzmq.so.3"
    #     dest: "/usr/local/lib/libzmq.so"
    #     state: "link"

    - name: Installing dumb-init
      apt:
        deb: "https://github.com/Yelp/dumb-init/releases/download/v1.0.2/dumb-init_1.0.2_amd64.deb"

    - name: Installing gosu
      get_url:
        url: "https://github.com/tianon/gosu/releases/download/{{ gosu_ver }}/gosu-amd64"
        dest: "/usr/local/bin/gosu"
        mode: 0755

    - name: Installing JRE
      apt:
        name: "default-jre-headless"
        state: "present"

    - name: Adding Elasticsearch Repo Key
      apt_key:
        url: "https://packages.elastic.co/GPG-KEY-elasticsearch"
        state: "present"

    - name: Adding Logstash Repo
      apt_repository:
        repo: "deb https://packages.elastic.co/logstash/{{ logstash_ver }}/debian stable main"

    - name: Installing Logstash
      apt:
        name: "logstash"
        state: "present"
        install_recommends: no

# We are doing this to redirect TCP/514 and UDP/514 to Logstash on TCP/10514
    - name: Configuring Rsyslogd 50-default.conf
      lineinfile:
        line: "*.* @@127.0.0.1:10514"
        dest: "/etc/rsyslog.d/50-default.conf"

# We are doing this to enable Rsyslogd to listen on tcp/514 and udp/514
    - name: Configuring Rsyslogd To Listen On TCP/514 and UDP/514
      lineinfile:
        dest: "/etc/rsyslog.conf"
        line: "{{ item }}"
      with_items:
        - '$ModLoad imudp'
        - '$UDPServerRun 514'
        - '$ModLoad imtcp'
        - '$InputTCPServerRun 514'
```

{% endraw %}

`logstash.conf`:

```json
input {
  beats {
    port => 5044
  }
}

input {
  tcp {
    type => "syslog"
    port => "10514"
  }
}

input {
  udp {
    type => "syslog"
    port => "10514"
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
  }
}
```

[**Kibana**](https://github.com/mrlesmithjr/docker-ansible-kibana)

`Dockerfile`:

```bash
FROM ubuntu:14.04

MAINTAINER Larry Smith Jr. <mrlesmithjr@gmail.com>

# Update apt-cache
RUN apt-get update

# Install Ansible
RUN apt-get -y install git software-properties-common && \
    apt-add-repository ppa:ansible/ansible && \
    apt-get update && \
    apt-get -y install ansible

# Copy Ansible Playbook
COPY playbook.yml /playbook.yml

# Run Ansible playbook
RUN ansible-playbook -i "localhost," -c local /playbook.yml

# Cleanup
RUN apt-get -y clean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PATH /opt/kibana/bin:$PATH

# Setup entrypoint Ansible Playbook
COPY docker-entrypoint.yml /docker-entrypoint.yml

# Setup entrypoint script
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]

# Expose port(s)
EXPOSE 5601

# Container start-up
CMD ["kibana"]
```

`playbook.yml`:
{% raw %}

```yaml
---
- hosts: localhost
  connection: local
  become: true
  vars:
    - gosu_ver: '1.9'
    - kibana_ver: '4.5'
    - tini_ver: 'v0.9.0'
  tasks:
    - name: Installing apt-transport-https
      apt:
        name: "apt-transport-https"
        state: "present"

    - name: Installing ca-certificates
      apt:
        name: "ca-certificates"
        state: "latest"

    - name: Installing dumb-init
      apt:
        deb: "https://github.com/Yelp/dumb-init/releases/download/v1.0.2/dumb-init_1.0.2_amd64.deb"

    - name: Installing gosu
      get_url:
        url: "https://github.com/tianon/gosu/releases/download/{{ gosu_ver }}/gosu-amd64"
        dest: "/usr/local/bin/gosu"
        mode: 0755

    - name: Installing tini
      get_url:
        url: "https://github.com/krallin/tini/releases/download/{{ tini_ver }}/tini"
        dest: "/usr/local/bin/tini"
        mode: 0755

    - name: Adding Elasticsearch Repo Key
      apt_key:
        url: "https://packages.elastic.co/GPG-KEY-elasticsearch"
        state: "present"

    - name: Adding Kibana Repo
      apt_repository:
        repo: "deb http://packages.elastic.co/kibana/{{ kibana_ver }}/debian stable main"

    - name: Installing Kibana
      apt:
        name: "kibana"
        state: "present"
        install_recommends: no

    - name: Ensuring Kibana Folder Permissions
      file:
        path: "/opt/kibana"
        owner: "kibana"
        group: "kibana"

    - name: Setting Default elasticsearch.url
      replace:
        dest: "/opt/kibana/config/kibana.yml"
        regexp: "^# elasticsearch.url: \"http://localhost:9200\""
        replace: "elasticsearch.url: \"http://elasticsearch:9200\""
```

{% endraw %}
**Using docker-compose**

And finally if we wanted to spin up the whole stack using docker-compose
we can do that by creating the following **_docker-compose.yml_** file:

```yaml
version: '2'
services:
  elasticsearch:
    image: mrlesmithjr/elasticsearch:latest
    volumes:
      - "./.data:/usr/share/elasticsearch/data"
    ports:
      - "9200:9200"
      - "9300:9300"
    restart: always

  kibana:
    depends_on:
      - elasticsearch
    image: mrlesmithjr/kibana:latest
    links:
      - elasticsearch
    ports:
      - "5601:5601"
    restart: always

  logstash:
    depends_on:
      - elasticsearch
    image: mrlesmithjr/logstash:latest
    links:
      - elasticsearch
    ports:
      - "5044:5044"
      - "10514:10514"
      - "10514:10514/udp"
    restart: always
```

And then spin it all up:

```bash
docker-compose up -d
```

Now fire up your browser of choice and head over to
<http://127.0.0.1:5601>.

And there you have it, you now have a containerized ELK Stack ready for
you to get creative with your GROKing.

**Creating your own Logstash configuration**

If you would like to change the included **_logstash.conf_** with your
own you could create your own Logstash configurations and place them in
the config folder and recreate your own **_Dockerfile_** as such for
example:

`Dockerfile`:

```bash
FROM mrlesmithjr/logstash:latest

COPY config/ /etc/logstash/conf.d

EXPOSE 3515 10514 10514/udp
```

`config/logstash.conf`:

```json
input {
  tcp {
    type => "syslog"
    port => "10514"
  }
}

input {
  udp {
    type => "syslog"
    port => "10514"
  }
}

input {
  tcp {
    type => "eventlog"
    port => "3515"
    codec => "json"
  }
}

output {
  elasticsearch {
    hosts =>; ["http://elasticsearch:9200"]
  }
}
```

Now you are ready to build your new Docker container for Logstash:

```bash
docker build -t mycustom/logstash .
```

And when that completes spin it up:

```bash
docker run -d -p 3515:3515 -p 10514:10514 -p 10514:10514/udp mycustom/logstash
```

Now enjoy your new custom Logstash configuration but don't let your
customization stop there. Head over to my GitHub repo
[here](https://github.com/mrlesmithjr/ansible-elk-processors/tree/master/templates/etc/logstash/conf.d)
to get more ideas on your parsing and etc.

Hope you have enjoyed this post and I definitely look forward to any
comments, thoughts, or opinions.

Enjoy!
