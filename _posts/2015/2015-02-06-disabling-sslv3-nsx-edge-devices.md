---
  title: Disabling SSLv3 on NSX Edge Devices
---

So a few months back after the Poodle attack I had a request to disable
SSLv3 on the NSX Edge Gateways so I started looking around and didn't
find anyway to do this other than disabling SSL versions for SSLVPN
therefore I had to setup the Edge LB to do SSL passthrough to the
webserver(s) themselves. Well I am here to let you know that this has
now been addressed and the original KB was updated yesterday to allow
disabling SSLv3 on an NSX Edge Gateway. The KB article can be read
[here](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2105096#sf36090735).
And the extracted portion pertaining to doing this is below.

> **"**
>
> **Disable SSLv3 Support on NSX Edge Load Balancer SSL Offload
> Services**:
>
> 1.  In the Load Balancer section of the NSX Edge configuration, create
>     an Application Profile.
> 2.  Restrict the ciphers to a trusted list.
>
> For more information on setting the cipher line, see See the following
> [openssl.org](http://www.openssl.org/docs/apps/ciphers.html) page.
>
> The Default is AES:ALL:!aNULL:!eNULL:+RC4:@STRENGTH\
> Sample cipher rule to disable
> SSLv3:AES:ALL:!aNULL:!eNULL:+RC4:@STRENGTH:!SSLv3:ALL
>
> **Disable SSLv3 Support on NSX Edge Load Balancer SSL Passthrough
> Services**:
>
> Currently with SSL Passthrough services, there is no functionality to
> restrict SSLv3. You must disable SSLv3 on the member pool systems of
> the load balancer VIP. Future releases will address this.
>
> "
