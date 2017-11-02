---
  title: HAProxy and MySQL Checks
  date: 2014-12-17
---

I wanted to throw this out in case anyone else has a need for such a
setup to use L7 for node up/down when load balancing MySQL with
HAProxy. All you have to do is run the following script. Copy the content below
into _setup_chkmysql.sh_

```bash
nano setup_chkmysql.sh
```

Copy contents from below, save and exit.

```bash
chmod +x setup_chkmysql.sh
sudo ./setup_chkmysql.sh
```

`haproxy-mysqlchk`:

```raw
#!/bin/bash
# This is for Ubuntu
apt-get install xinetd

(
cat << 'EOF'
# default: on
# description: mysqlchk
service mysqlchk
{
        flags           = REUSE
        socket_type     = stream
        port            = 9200
        wait            = no
        user            = nobody
        server          = /usr/local/bin/mysqlchk.sh
        log_on_failure  += USERID
        disable         = no
#        only_from       = 0.0.0.0/0
# recommended to put the IPs that need
# to connect exclusively (security purposes)
        per_source      = UNLIMITED
# Recently added (May 20, 2010)
# Prevents the system from complaining
# about having too many connections open from
# the same IP. More info:
# http://www.linuxfocus.org/English/November2000/article175.shtml
}
EOF
) | tee /etc/xinetd.d/mysqlchk


/usr/sbin/update-rc.d xinetd defaults
echo "mysqlchk 9200/tcp" >> /etc/services

#nano usr/local/bin/mysqlchk.sh
(
cat << 'EOF'
#!/bin/bash
#
# This script checks if a mysql server is healthy running on localhost. It will
# return:
# "HTTP/1.x 200 OK\r" (if mysql is running smoothly)
# - OR -
# "HTTP/1.x 500 Internal Server Error\r" (else)
#
# The purpose of this script is make haproxy capable of monitoring mysql properly
#

MYSQL_HOST="127.0.0.1"
MYSQL_PORT="3306"
MYSQL_USERNAME="cmon"
MYSQL_PASSWORD="cmon"
MYSQL_OPTS="-N -q -A"
TMP_FILE="/dev/shm/mysqlchk.$$.out"
ERR_FILE="/dev/shm/mysqlchk.$$.err"
FORCE_FAIL="/dev/shm/proxyoff"
MYSQL_BIN="/usr/bin/mysql"
CHECK_QUERY="show global status where variable_name='wsrep_local_state'"
preflight_check()
{
    for I in "$TMP_FILE" "$ERR_FILE"; do
        if [ -f "$I" ]; then
            if [ ! -w $I ]; then
                echo -e "HTTP/1.1 503 Service Unavailable\r\n"
                echo -e "Content-Type: Content-Type: text/plain\r\n"
                echo -e "\r\n"
                echo -e "Cannot write to $I\r\n"
                echo -e "\r\n"
                exit 1
            fi
        fi
    done
}
return_ok()
{
    echo -e "HTTP/1.1 200 OK\r\n"
    echo -e "Content-Type: text/html\r\n"
    echo -e "Content-Length: 43\r\n"
    echo -e "\r\n"
    echo -e "<html><body>MySQL is running.</body></html>\r\n"
    echo -e "\r\n"
    rm $ERR_FILE $TMP_FILE
    exit 0
}
return_fail()
{
    echo -e "HTTP/1.1 503 Service Unavailable\r\n"
    echo -e "Content-Type: text/html\r\n"
    echo -e "Content-Length: 42\r\n"
    echo -e "\r\n"
    echo -e "<html><body>MySQL is *down*.</body></html>\r\n"
    sed -e 's/\n$/\r\n/' $ERR_FILE
    echo -e "\r\n"
    rm $ERR_FILE $TMP_FILE
    exit 1
}
preflight_check
if [ -f "$FORCE_FAIL" ]; then
        echo "$FORCE_FAIL found" > $ERR_FILE
        return_fail;
fi
$MYSQL_BIN $MYSQL_OPTS --host=$MYSQL_HOST --port=$MYSQL_PORT --user=$MYSQL_USERNAME --password=$MYSQL_PASSWORD -e "$CHECK_QUERY" > $TMP_FILE 2> $ERR_FILE
if [ $? -ne 0 ]; then
        return_fail;
fi
status=`cat  $TMP_FILE | awk '{print $2;}'`

if [ $status -ne 4 ]; then
   return_fail;
fi

return_ok;
EOF
) | tee /usr/local/bin/mysqlchk.sh

chmod 777 /usr/local/bin/mysqlchk.sh
/etc/init.d/xinetd restart
```

Now add the following for you haproxy.cfg file on your load balancer.

```bash
frontend mysql-3306
        bind 192.168.1.71:3306
        mode tcp
        log global
        option tcplog
        timeout client 60000ms
        default_backend mysql_tcp

backend mysql_tcp
        balance leastconn
        timeout connect 60000ms
        timeout server 60000ms
        option httpchk
        option allbackups
        default-server port 9200 inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 256 maxqueue 128 weight 100
        server galera-1 galera-1.everythingshouldbevirtual.local:3306 check
        server galera-2 galera-2.everythingshouldbevirtual.local:3306 check
        server galera-3 galera-3.everythingshouldbevirtual.local:3306 check
```

Enjoy!
