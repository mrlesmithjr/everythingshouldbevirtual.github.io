---
  title: Graylog2 v0.92.4 Install Script
---

This post will only cover a fresh new install of Graylog2.

To install Graylog2 v0.92.4 do the following on a Ubuntu 12.x/13.x/14.x server.

```bash
sudo apt-get install git
cd ~
git clone https://github.com/mrlesmithjr/graylog2
chmod +x graylog2/install_graylog2_90_ubuntu.sh
sudo ./graylog2/install_graylog2_90_ubuntu.sh
```

For additional steps in setting up head over to [this](https://everythingshouldbevirtual.com/ubuntu-12-04-graylog2-installation) post.

Enjoy!
