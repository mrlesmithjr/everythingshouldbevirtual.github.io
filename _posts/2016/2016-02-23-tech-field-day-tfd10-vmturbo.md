---
  title: "Tech Field Day - #TFD10 - VMTurbo"
---

Once again [VMTurbo](http://vmturbo.com/) presented at Tech Field Day
and this time it was at #TFD10 in Austin, TX. VMTurbo is known for
their unique way of managing your virtual and cloud environments. They
utilize a supply and demand model to abstract the workloads in your
environment(s) which includes VMs, containers, compute, network, storage
and etc. Which when you listen to their story it is very business
(economics) oriented model, meaning that their product is easy to
consume from a non-technical level. But for me personally I get caught
up in trying to keep up with all of the economic jargon and find myself
trying to pull myself out of my technical mindset and listen from a
different angle. (Hope that all made sense :) )

VMTurbo evaluates your environment holistically and begins to manipulate
elements to properly balance out resources. This is much different than
from a VMWare DRS/SDRS perspective of just moving workloads around to
balance storage and compute resources. I know that many have questioned
in the past on why they would ever use VMTurbo past a 30-day evaluation
because at that point their environment will be in much better shape and
the need for VMTurbo is then non-existent. That obviously is not true
because VMTurbo's purpose is to not identify issues within your
environment but rather to better balance your environment so that it is
continually in a ideal state. This approach allows for the tool to
automate most if not all of the manual tasks that you may today perform
and removes the guess-work. Again because of their supply and demand
economical terminologies you may truly get lost in the weeds if you are
extremely technical. So let me explain some things that this product is
capable of which are consumable as technical details.

VMTurbo can automate and make decisions on placement and reallocation of
a workloads objects for it's entire lifecycle. Again this could include
compute resources (memory and cpu),
[fabrics](http://vmturbo.com/product/control-modules/ucs-fabric-management-software/),
cluster resource placement, [network
resources](http://vmturbo.com/product/control-modules/network-management-software/)
and/or [storage
resources](http://vmturbo.com/product/control-modules/storage-resource-management/).
VMTurbo works for you in your private cloud, public cloud, [hybrid
cloud](http://vmturbo.com/product/control-modules/hybrid-cloud-management-software/)
and even multi cloud environments. VMTurbo supports numerous hypervisors
including, Hyper-V, KVM, VMware and XEN. As well as the ability to
support [Openstack](http://vmturbo.com/solutions/use-cases/openstack/)
and
[containers](http://vmturbo.com/product/control-modules/container-management-software/).

Now for some of the coolness that is not discussed much about VMTurbo in
my opinion is the ability to leverage VMTurbo as an entry-point
leveraging API's as part of your automated workflow pipeline. How about
having your automation tooling which may trigger a Jenkins job that
spins up nodes in your cloud by just talking directly to the VMTurbo API
and let VMTurbo select the proper placement for your workload(s)? Pretty
cool right? The options with this are endless to say the least.

But here are some things that I would like to see brought to the product
(assuming they are not already working on these :) ). How cool would it
be to have VMTurbo not only keep your applications and workloads in an
ideal state, but to actually take it a step further and auto-scale your
workloads based on your defined desired availability and utilization.
Now that would be cool right? But what about being able to even take
into account Geo-Location and either move workloads based on demand in a
region or even spin up new workloads in those regions. Hmmmm. I mean,
VMTurbo already has all of the data that is required to make those
decisions so why not take this product to the next level? And again
these scenarios could be endless and take forever putting into words but
all I can say is keep a look out for more cool things coming from
VMTurbo.

> Disclaimer:
> _All meals, travel and entertainment was provided by Gestalt IT. However
> Gestalt IT nor the Vendor have provided any type of compensation to
> write-up any portion of this article. The information contained within
> this article are solely my views and take aways._
