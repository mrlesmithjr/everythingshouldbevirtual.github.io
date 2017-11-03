---
  title: Nginx Reverse Proxy
  date: 2013-04-22 17:06:32
---

I just wanted to throw this together real quick as I had a request this
past week to look into this. I was asked to look into deploying a
reverse proxy for a DMZ scenario with backend web servers not in the
DMZ. So after spending a good bit of time looking into the Microsoft ARR
solution and not having great luck or any consistency I finally turned
to Linux. :) I first started with a squid reverse proxy and then decided
to look at the Nginx reverse proxy. I would have to say it was much
easier and straight forward to configure and get up and running fairly
quick. And it runs great.

First thing you will need to do is install nginx.

```bash
sudo apt-get install nginx
```

Now you will need to create a conf file which will contain the info of
the backend servers you want to configfure for the reverse proxy. I am
running the reverse proxy here on tcp port 8080. You can set this to 80
or whatever other port you may want to use. You can create the
proxy.conf file info from below to get your started.

```bash
 nano /etc/nginx/conf.d/proxy.conf
```

```bash
server {
 listen 8080;
 server_name whateveryourdomain.com;
 access_log off;
 error_log off;
 location / {
 proxy_pass http://backendserverip/;
 proxy_redirect off;
 proxy_set_header Host $host;
 proxy_set_header X-Real-IP $remote_addr;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_max_temp_file_size 0;
 client_max_body_size 10m;
 client_body_buffer_size 128k;
 proxy_connect_timeout 90;
 proxy_send_timeout 90;
 proxy_read_timeout 90;
 proxy_buffer_size 4k;
 proxy_buffers 4 32k;
 proxy_busy_buffers_size 64k;
 proxy_temp_file_write_size 64k;
 }
}
```

Now restart nginx for the config you created above to take affect.

```bash
sudo /etc/init.d/nginx restart
```

Now you will need to create a NAT/Port Forward rule on your firewall to
pass all tcp port 80 to your nginx reverse proxy which in this case is 8080.
So it will be like tcp/80 --> tcp/8080.

You should now be able to test that all is working. If all goes well then you
should be good to go.

For ISPConfig setups to get stats reporting correctly you will need to make the
following change.

/etc/apache2/sites-available/ispconfig.conf

```bash
LogFormat "%v %h %l %u %t \"%r\" %>s %B \"%{Referer}i\" \"%{User-Agent}i\"" combined_ispconfig
```

To

```bash
LogFormat "%v %{X-Real-IP}i %l %u %t \"%r\" %>s %B \"%{Referer}i\" \"%{User-Agent}i\"" combined_ispconfig
```

This is all for now and I will update this post as needed going forward.
Feel free to leave any comments or suggestions!

Enjoy!
