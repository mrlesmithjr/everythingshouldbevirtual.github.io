---
title: "Kubernetes - Upgrading Cluster To 1.10 - Part 1"
date: "2018-04-05 08:46"
categories:
  - Containers
tags:
  - Kubernetes
---

> NOTE: This post assumes that a Debian based OS is being used.

Now that I have been running my [Raspberry Pi Kubernetes Cluster](http://everythingshouldbevirtual.com/automation/containers/ansible-raspberry-pi-kubernetes-cluster/) for a few months, I felt it was time to go through an actual Kubernetes upgrade.
I initially wanted to automate this process but I am still trying to sort out
an easy solution to this. So for now I will just be going through and doing this
manually using `kubeadm`.

## Planning Cluster Upgrade

Before you proceed onto upgrading your cluster you will need to properly plan out
your upgrade to ensure that you have met all pre-reqs. Luckily for us, the `kubeadm`
tool provides us the ability to properly plan out the upgrade process. So if we
take a look at what `kubeadm` provides us when we run the following:

```bash
sudo kubeadm upgrade plan
```

And we should get some interesting results back from the above:

```bash
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.9.3
[upgrade/versions] kubeadm version: v1.9.3
[upgrade/versions] Latest stable version: v1.10.0
[upgrade/versions] Latest version in the v1.9 series: v1.9.6

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT      AVAILABLE
Kubelet     5 x v1.9.3   v1.9.6

Upgrade to the latest version in the v1.9 series:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.9.3    v1.9.6
Controller Manager   v1.9.3    v1.9.6
Scheduler            v1.9.3    v1.9.6
Kube Proxy           v1.9.3    v1.9.6
Kube DNS             1.14.7    1.14.7
Etcd                 3.1.11    3.1.11

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.9.6

Note: Before you can perform this upgrade, you have to update kubeadm to v1.9.6.

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT      AVAILABLE
Kubelet     5 x v1.9.3   v1.10.0

Upgrade to the latest stable version:

COMPONENT            CURRENT   AVAILABLE
API Server           v1.9.3    v1.10.0
Controller Manager   v1.9.3    v1.10.0
Scheduler            v1.9.3    v1.10.0
Kube Proxy           v1.9.3    v1.10.0
Kube DNS             1.14.7    1.14.7
Etcd                 3.1.11    3.1.11

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.10.0

Note: Before you can perform this upgrade, you have to update kubeadm to v1.10.0.

_____________________________________________________________________
```

And if you look at the results from the above you will notice that we have been
provided with the steps in which we will need to follow in order to get our cluster
upgraded. And based on my results it appears that we are currently running `v1.9.3`
and `v1.10.0` is available. But we must upgrade to `v1.9.6` first before we can
get to `v1.10.0`.

## Upgrading Cluster

If we follow the upgrade steps from the above we now have a plan to upgrade our
cluster. So let's hope that this goes well.

### Upgrading To v1.9.6

Based on our [Planning Cluster Upgrade](#planning-cluster-upgrade) we were told
that we need to first upgrade our cluster to `v1.9.6` so let's do that and see
how everything turns out.

Our instructions said to run the following:

```bash
kubeadm upgrade apply v1.9.6
```

And immediately we are presented with the following:

```raw
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/version] You have chosen to change the cluster version to "v1.9.6"
[upgrade/versions] Cluster version: v1.9.3
[upgrade/versions] kubeadm version: v1.9.3
[upgrade/version] FATAL: The --version argument is invalid due to these errors:

	- Specified version to upgrade to "v1.9.6" is higher than the kubeadm version "v1.9.3". Upgrade kubeadm first using the tool you used to install kubeadm

Can be bypassed if you pass the --force flag
```

So the first thing I must do here is do the following to resolve the above:

```bash
sudo apt-get update
sudo apt-get install kubeadm
```

Now we are ready to try again to upgrade:

```raw
sudo kubeadm upgrade apply v1.9.6
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/config] FATAL: could not decode configuration: unable to decode config from bytes: v1alpha1.MasterConfiguration: KubeProxy: v1alpha1.KubeProxy: Config: v1alpha1.KubeProxyConfiguration: FeatureGates: ReadMapCB: expect { or n, but found ", error found in #10 byte of ...|reGates":"","healthz|..., bigger context ...|24h0m0s"},"enableProfiling":false,"featureGates":"","healthzBindAddress":"0.0.0.0:10256","hostnameOv|...
```

And guess what? Another error has presented itself here. So after a bit of Google
searching I came across [this issue](https://github.com/kubernetes/kubernetes/issues/61764)
which states changing `featureGates: ""` to `featureGates: {}` by doing the following:

```bash
kubectl -n kube-system edit cm kubeadm-config
```

And once again we try the upgrade again:

```raw
sudo kubeadm upgrade apply v1.9.6
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[upgrade/version] You have chosen to change the cluster version to "v1.9.6"
[upgrade/versions] Cluster version: v1.9.3
[upgrade/versions] kubeadm version: v1.10.0
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Will prepull images for components [kube-apiserver kube-controller-manager kube-scheduler]
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.9.6"...
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests213337574/etcd.yaml"
[certificates] Generated etcd/ca certificate and key.
[certificates] Generated etcd/server certificate and key.
[certificates] etcd/server serving cert is signed for DNS names [localhost] and IPs [127.0.0.1]
[certificates] Generated etcd/peer certificate and key.
[certificates] etcd/peer serving cert is signed for DNS names [rpi-k8s-1] and IPs [192.168.100.1]
[certificates] Generated etcd/healthcheck-client certificate and key.
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests677733901/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests213337574"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests213337574/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests213337574/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests213337574/kube-scheduler.yaml"
[certificates] Using the existing etcd/ca certificate and key.
[certificates] Generated apiserver-etcd-client certificate and key.
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests677733901/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/apply] FATAL: couldn't upgrade control plane. kubeadm has tried to recover everything into the earlier state. Errors faced: [timed out waiting for the condition]
```

And once again another failure! So I attempt the upgrade once again and here we
are!

```raw
sudo kubeadm upgrade apply v1.9.6
[preflight] Running pre-flight checks.
[upgrade] Making sure the cluster is healthy:
[upgrade/health] FATAL: [preflight] Some fatal errors occurred:
	[ERROR APIServerHealth]: the API Server is unhealthy; /healthz didn't return "ok"
	[ERROR MasterNodesReady]: couldn't list masters in cluster: Get https://192.168.100.1:6443/api/v1/nodes?labelSelector=node-role.kubernetes.io%2Fmaster%3D: http2: server sent GOAWAY and closed the connection; LastStreamID=1, ErrCode=NO_ERROR, debug=""
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```

And now the cluster appears to be hosed!

```bash
kubectl get all --all-namespaces
error: {batch  cronjobs} matches multiple kinds [batch/v1beta1, Kind=CronJob batch/v2alpha1, Kind=CronJob]
```

After rebooting all of the hosts within the cluster everything seems to be
broken still. Isn't this fun!

And [this](https://github.com/kubernetes/kubeadm/issues/740) open issue seems
like it may be related.

And at this point it seems like I may need to reset the cluster back to default
or do a complete rebuild!
