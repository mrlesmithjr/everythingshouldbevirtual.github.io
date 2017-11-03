---
  title: Highly Available ELK (Elasticsearch, Logstash and Kibana) Unicast Mode
  date: 2014-06-26
---

In the previous [post](https://everythingshouldbevirtual.com/highly-available-elk-elasticsearch-logstash-kibana-setup "Highly Available ELK (Elasticsearch, Logstash and Kibana) Setup")
we setup the complete ELK stack from top to bottom. And everything should
be working as needed; however one thing you will want to do once
everything is stable is to change all ES (Elasticsearch) nodes to
communicate in unicast mode instead of how it was setup which was in
mulitcast discovery mode. Switching to unicast mode will keep all of the
network chatter down doing discovery of your ES cluster. This will also
result in better performance for your cluster. I have made all of this
extremely simple for you if you followed the previous post and used the
install scripts. I have included all of the settings that will need to
be changed in all of the configuration files so all that is required is
to comment out certain sections, uncomment the relevant sections
required and restart all services.

So to switch over to unicast mode we will need to do the following.

**ES (Elasticsearch Master/Data Nodes (es-1, es-2):**

On each of your ES Master/Data nodes from a terminal session change the
following settings. (Remember that if you used different hostnames/IPs
your settings will be different)

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Now at the bottom of the config file you will see the following.

```bash
##### Uncomment below instead of using multicast and update with your actual ES Master/Data nodenames #####
#discovery.zen.ping.unicast.hosts: ["es-1", "es-2"]
#discovery.zen.ping.multicast.enabled: false
```

You will need to uncomment these lines out so they look like the
following.

```bash
##### Uncomment below instead of using multicast and update with your actual ES Master/Data nodenames #####
discovery.zen.ping.unicast.hosts: ["es-1", "es-2"]
discovery.zen.ping.multicast.enabled: false
```

Now you will need to restart the elasticsearch service

```bash
sudo service elasticsearch restart
```

And if all went well your cluster should be back up and running. You can
verify and look for any issues by checking the elasticsearch log.

```bash
tail /var/log/elasticsearch/logstash-cluster.log
```

You should see successful discovery of your cluster and an elected
master node. So if all went well you will now need to proceed to each of your ELK
frontend nodes.

**ELK (Elasticsearch, Logstash and Kibana) Nodes (logstash-1, logstash-2):**

On each of your ELK frontend nodes from a terminal session change the
following settings.

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Now at the bottom of the config file you will see the following.

```bash
##### Uncomment below instead of using multicast and update with your actual ES Master/Data nodenames #####
#discovery.zen.ping.unicast.hosts: ["es-1", "es-2"]
#discovery.zen.ping.multicast.enabled: false
```

You will need to uncomment these lines out so they look like the
following.

```bash
##### Uncomment below instead of using multicast and update with your actual ES Master/Data nodenames #####
discovery.zen.ping.unicast.hosts: ["es-1", "es-2"]
discovery.zen.ping.multicast.enabled: false
```

Now you will need to restart the elasticsearch service

```bash
sudo service elasticsearch restart
```

Now you will need to modify your logstash.conf file to join the ES
_"logstash-cluster"_ cluster in unicast mode.\
In order to this you will do the following.

```bash
sudo nano /etc/logstash/logstash.conf
```

You will see the following at the very bottom of the config file.

```json
#### Multicast discovery mode ####
# Send output to the ES cluster logstash-cluster using a predefined template
# The following settings will be used during the initial setup which will be used for using multicast ES nodes
# When changing to unicast discovery mode you need to comment out the following section and configure the unicast discovery mode in the next section
output {
        elasticsearch {
                cluster => "logstash-cluster"
                flush_size => 1
                manage_template => true
                template => "/opt/logstash/lib/logstash/outputs/elasticsearch/elasticsearch-template.json"
        }
}
```

```json
#### Unicast discovery mode ####
# Send output to the ES cluster logstash-cluster using a predefined template
# The settings below will be used when you change to unicast discovery mode for all ES nodes
# Make sure to comment out the above multicast discovery mode section
#output {
#        elasticsearch {
#                cluster => "logstash-cluster"
#                host => "logstash"
#                port => "9300"
#                protocol => "node"
#                flush_size => "1"
#                manage_template => true
#                template => "/opt/logstash/lib/logstash/outputs/elasticsearch/elasticsearch-template.json"
#        }
#}
```

Now change these lines to look like the following.

```json
#### Multicast discovery mode ####
# Send output to the ES cluster logstash-cluster using a predefined template
# The following settings will be used during the initial setup which will be used for using multicast ES nodes
# When changing to unicast discovery mode you need to comment out the following section and configure the unicast discovery mode in the next section
#output {
#        elasticsearch {
#                cluster => "logstash-cluster"
#                flush_size => 1
#                manage_template => true
#                template => "/opt/logstash/lib/logstash/outputs/elasticsearch/elasticsearch-template.json"
#        }
#}

#### Unicast discovery mode ####
# Send output to the ES cluster logstash-cluster using a predefined template
# The settings below will be used when you change to unicast discovery mode for all ES nodes
# Make sure to comment out the above multicast discovery mode section
output {
        elasticsearch {
                cluster => "logstash-cluster"
                host => "logstash"
                port => "9300"
                protocol => "node"
                flush_size => "1"
                manage_template => true
                template => "/opt/logstash/lib/logstash/outputs/elasticsearch/elasticsearch-template.json"
        }
}
```

Once you have changed these settings you need to restart the logstash
service.

```bash
sudo service logstash restart
```

You should now have all ELK components communicating via unicast instead
of multicast now. If you have any issues with communications from your
logstash services make sure that you have the following settings in your
HAProxy nodes configuration. Again if you followed the initial setup in
the previous post these settings should already be there.

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

You should see the following in this configuration file.

```bash
listen elasticsearch-TCP-9300 10.0.101.60:9300
        mode tcp
        option tcpka
        option tcplog
        #balance leastconn - The server with the lowest number of connections receives the connection
        #balance roundrobin - Each server is used in turns, according to their weights.
        #balance source - Source IP hashed and divided by total weight of servers designates which server will receive the request
        balance roundrobin
        server es-1 es-1:9300 check
        server es-2 es-2:9300 check
```

If this section is missing you will need to add it and reload your
haproxy config.

```bash
sudo service haproxy reload
```

So once all of this has been setup you should be good to go running in
Unicast mode for your whole ES cluster. This will also allow you to run
your different nodes outside of the same subnet as well as allow you to
add additional nodes in different subnets.

Enjoy!
