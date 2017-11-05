---
  title: Cisco ASA Logstash Parsing
  date: 2014-12-11
---

I recently had an opportunity to get around to creating some Cisco ASA
parsing for logstash to detect some abnormal activity on the network. So
now that I have created the parsing and have to say it works pretty
good; I figured I would share it with anyone else that may have a need
for this as well. I will also share the dashboard that I created in case
you want that as well.

You will need to add the following to your current logstash.conf file..
I placed this at the top of my config below all inputs before standard
syslog parsing to make sure it was processed first, tagged and passed
the next level of syslog parsing.
{% gist mrlesmithjr/791dc72d3c92ac21342f %}

That's it!

Here are some screenshots of what the dashboard looks like.

![Screen Shot 2014-12-11 at 9.58.47 PM](../../assets/Screen-Shot-2014-12-11-at-9.58.47-PM-300x145.png)

![Screen Shot 2014-12-11 at 9.59.05 PM](../../assets/Screen-Shot-2014-12-11-at-9.59.05-PM-300x54.png)

Below is the dashboard code:
{% gist mrlesmithjr/a143d71deca65c9b64e7 %}

Enjoy!
