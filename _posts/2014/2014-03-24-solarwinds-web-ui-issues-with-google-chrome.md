---
title: Solarwinds Web UI Issues with Google Chrome
date: 2014-03-24 07:00:45
---

So in the past few days, I have witnessed on two different environments
when trying to connect to certain components of Solarwinds using Google
Chrome you will see the following.

![19-19-04](../../assets/19-19-04-300x179.png)

Obviously this is a pain because the page will not display. So after
opening the same page using Firefox, you will see the following.

![19-29-34](../../assets/19-29-34-300x141.png)

So it appears that the issue is with the SSL cert connecting back into
these components. In this case I am trying to view Virtualization
Manager maps. So in order to get around this we need to find the IP
address of the Virtualization Manager appliance so we can accept the
untrusted SSL certificate.

In order to find the Virtualization Manager IP address you can go the
_settings_ link at the top of your Solarwinds Web UI page.

![19-35-23](../../assets/19-35-23-300x12.png)

Now click on _Virtualization settings_.

![19-33-35](../../assets/19-33-35-300x78.png)

Now click on _setup virtualization manager integration_.

![19-38-30](../../assets/19-38-30-300x68.png)

The IP address for Virtualization manager will be listed as below.

![19-44-35](../../assets/19-44-35-300x229.png)

Now go to <https://virtualizationmanagerIPorFQDN> and select proceed
anyway.

![20-13-11](../../assets/20-13-11-300x81.png)

You should now see the following.

![20-16-36](../../assets/20-16-36-300x142.png)

Now head back to your Solarwinds Web UI and go to Virtualization maps
(this example) and you should now be presented with a login window now
instead of the very first screenshot in this post.

![20-18-31](../../assets/20-18-31-300x142.png)

That's it...You should be good to go now.

Enjoy!
