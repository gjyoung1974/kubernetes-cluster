# KVM2 Kubernetes cluster

- Set up a very basic local Kubernetes cluster using Kubeadm
- For testing k8s infra components
- Hosted on KVM2/Centos

## Pre-requisites

 * **[Vagrant](https://www.vagrantup.com)**
 * **[libvirt](https://wiki.centos.org/HowTos/KVM)**

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
