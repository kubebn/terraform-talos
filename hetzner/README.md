# Terraform examples

Local utilities

* terraform
* talosctl
* kubectl
* yq

# Talos on Hetzner Cloud

This terraform example install Talos on [HCloud](https://www.hetzner.com/cloud) with IPv4/IPv6 support.

<img src="/img/hetzner.png" width="500px">

## Kubernetes addons

* [cilium](https://github.com/cilium/cilium) 1.10.0
* [kubelet-serving-cert-approver](https://github.com/alex1989hu/kubelet-serving-cert-approver)
* [metrics-server](https://github.com/kubernetes-sigs/metrics-server) 0.5.0
* [rancher.io/local-path](https://github.com/rancher/local-path-provisioner) 0.0.19
* [hcloud-cloud-controller-manage](https://github.com/sergelogvinov/hetzner-cloud-controller-manager) fork of [syself](https://github.com/syself/hetzner-cloud-controller-manager) with few changes

```sh
NAME           STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP       OS-IMAGE                         KERNEL-VERSION    CONTAINER-RUNTIME     ZONE         REGION
store-1        Ready    storage                206d    v1.24.3   172.16.2.51   65.21.XX.XX       Debian GNU/Linux 11 (bullseye)   5.10.0-15-amd64   containerd://1.4.13   hel1-dc1     hel1
master-1       Ready    control-plane,master   207d    v1.24.2   172.16.0.11   65.108.XX.XX      Talos (v1.1.1)                   5.15.54-talos     containerd://1.6.6    hel1-dc2     hel1
master-2       Ready    control-plane,master   206d    v1.24.2   172.16.0.12   159.69.XX.XX      Talos (v1.1.1)                   5.15.54-talos     containerd://1.6.6    fsn1-dc14    fsn1
master-3       Ready    control-plane,master   26h     v1.24.2   172.16.0.13   65.108.XX.XX      Talos (v1.1.1)                   5.15.54-talos     containerd://1.6.6    hel1-dc2     hel1
```

Where:
* master-X - talos control plane nodes
* store-X - debian bare metal worker servers

## Prepare the base image

Use packer (system_os/hetzner) to upload image.

## Create control plane lb

open config file **terraform.tfvars** and add params.

```hcl
# counts and type of kubernetes master nodes
controlplane = {
    count   = 1,
    type    = "cpx11"
    type_lb = ""
}

# regions to use
regions = ["nbg1", "fsn1", "hel1"]

# counts and type of worker nodes by redion
instances = {
    "nbg1" = {
      web_count            = 0,
      web_instance_type    = "cx11",
      worker_count         = 0,
      worker_instance_type = "cx11",
    },
    "fsn1" = {
      web_count            = 0,
      web_instance_type    = "cx11",
      worker_count         = 0,
      worker_instance_type = "cx11",
    }
    "hel1" = {
      web_count            = 1,
      web_instance_type    = "cx11",
      worker_count         = 1,
      worker_instance_type = "cx11",
    }
}
```

```shell
make create-lb
```

## Install control plane

Generate the default talos config

```shell
make create-config create-templates
```

And deploy the kubernetes master nodes

```shell
make create-controlplane
```

Bootstrap the first node

```shell
talosctl --talosconfig _cfgs/talosconfig config endpoint $controlplane_firstnode
talosctl --talosconfig _cfgs/talosconfig --nodes $controlplane_firstnode bootstrap
```

```shell
make create-kubeconfig
```

## Deploy all other instances

```shell
make create-infrastructure
```
