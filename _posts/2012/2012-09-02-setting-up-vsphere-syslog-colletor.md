---
  title: Setting up vSphere Syslog Colletor
  date: 2012-09-02 15:56:18
---

Here is a quick step by step to installing the syslog collector for
vSphere. This utility is part of the vCenter server installation media.
I will be installing it as part of the vCenter integrated type. The
installation is very straight forward, the only setting I would
recommend highly changing is the destination directory for the actual
log data to be stored. I would move this off of the C:\\. Once the
installation completes open the viclient and go to the home screen and
you will see the syslog collector now available. You will also need to
configure your hosts to point back to vcenter for syslog. This is done
through configuration, software advanced settings, syslog, global. Make
sure to look through the screenshots.
