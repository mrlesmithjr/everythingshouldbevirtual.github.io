---
  title: Graylog2 Streams via Email
  date: 2013-08-02 23:11:02
---

I was recently asked how to get emails working from streams you have
created within the Graylog2 web ui. Seeing as I had done this just
recently I thought I would share what I did to get them working.

So the first thing you need to do is modify /etc/graylog2.conf and find
the section # Email Transport and modify like below. That way you can
get emails to flowing from Graylog2.

```bash
nano /etc/graylog2.conf
# Email transport
transport_email_enabled = true
transport_email_protocol = smtp
transport_email_hostname = yoursmtpserver
transport_email_port = 25
transport_email_use_auth = false
transport_email_use_tls = false
transport_email_auth_username = you@example.com
transport_email_auth_password = secret
transport_email_subject_prefix = [graylog2]
transport_email_from_email = email@yourdomain.com
transport_email_from_name = Graylog2
transport_email_web_interface_url = http://yourgraylogservername.domain.com
```

Now create your streams however you want and set the thresholds.

![22-58-23](../../assets/22-58-23-300x60.png)

![22-58-59](../../assets/22-58-59-300x85.png)

![22-59-23](../../assets/22-59-23-300x96.png)

![22-59-54](../../assets/22-59-54-300x86.png)

![23-00-17](../../assets/23-00-17-300x210.png)

Make the alarm active and select I want to receive alarms. And set your
messages, minutes and grace period.

![23-07-14](../../assets/23-07-14-300x97.png)

Now edit your username and make sure that you have an email address
added for your user that you want to receive emails.

![23-02-03](../../assets/23-02-03-300x139.png)

That's it!

Enjoy!
