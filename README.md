# Libvirt (KVM2) Kubernetes cluster

- Set up a very basic local Kubernetes cluster using Kubeadm
- For testing k8s infra components
- Hosted on [libvirt](https://libvirt.org/): /(KVM2) on Centos     
    (should be portable to other distros)     
    
## TODO:
-   Port this from Vagrant to Terraform
[terraform-provider-libvirt](https://github.com/dmacvicar/terraform-provider-libvirt/tree/master/examples/v0.12/coreos)     

## This installs
 * **3 node k8s cluster (1x Master, 2x workers)**
 * **Standard compliment of k8 components, API, Controller, Scheduler, Proxy, Etcd, DNS, etc...**
 * **Ingress controller**
 * **Calico network policies**

## Pre-requisites

 * **[libvirt](https://wiki.centos.org/HowTos/KVM)**
 * **[Vagrant](https://www.vagrantup.com)**
 * **[vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)**
 ```sh
    $ vagrant plugin install vagrant-libvirt
```
* **[Centos/Fedora vagrant-libvirt](https://developer.fedoraproject.org/tools/vagrant/vagrant-libvirt.html)**

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
