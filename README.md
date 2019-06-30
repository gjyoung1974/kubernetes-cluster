# Libvirt (KVM2) Kubernetes cluster

- Set up a very basic local Kubernetes cluster using Kubeadm
- For testing k8s infra components
- Hosted on KVM2/(libvirt) on Centos
    (should be portable to other distros)     
    
## TODO:
-   Port this from Vagrant to Terraform
[terraform-provider-libvirt](https://github.com/dmacvicar/terraform-provider-libvirt/tree/master/examples/v0.12/coreos)     

## Pre-requisites

 * **[libvirt](https://wiki.centos.org/HowTos/KVM)**
 * **[Vagrant](https://www.vagrantup.com)**

## How to Run

Execute the following vagrant command to start a new Kubernetes cluster, this will start one master and two nodes:

```sh
sudo vagrant up --no-parallel
```

## Clean-up

Execute the following command to remove the virtual machines created for the Kubernetes cluster.

```sh
sudo vagrant destroy -f
```
