---
  title: Graylog2 v0.90 Install Script
  date: 2014-10-02
---

In the past I have been updating the auto install scripts for Graylog2
and updating the original post each time. However here on out I will be
creating new posts for each release in hopes of keeping it much cleaner.
I will be including a link back to the original post for reference on
setting up and additional information as well as comments from others.

So at this time I am releasing the latest install script for Graylog2
v0.90 on Ubuntu. Please provide any feedback.

This post will only cover a fresh new install of Graylog2 however I will
be working on upgrade scripts from v0.20 versions to the newest v0.90
and they will be in separate posts as well to keep confusion down.

To install Graylog2 v0.90 do the following on a Ubuntu 12.x/13.x/14.x
server.

```bash
sudo apt-get install git
cd ~
git clone https://github.com/mrlesmithjr/graylog2
chmod +x graylog2/install_graylog2_90_ubuntu.sh
sudo ./graylog2/install_graylog2_90_ubuntu.sh
```

For additional steps in setting up head over to
[this](http://everythingshouldbevirtual.com/ubuntu-12-04-graylog2-installation "Ubuntu 12.04 Graylog2 Installation")
post.

Enjoy!
