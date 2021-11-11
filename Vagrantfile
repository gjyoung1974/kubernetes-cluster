# -*- mode: ruby -*-
# vi: set ft=ruby :

servers = [
    {
        :name => "k8s-master",
        :type => "master",
        :box => "centos/7",
        :box_version => "1902.01",
        :eth1 => "192.168.100.50",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "k8s-node01",
        :type => "node",
        :box => "centos/7",
        :box_version => "1902.01",
        :eth1 => "192.168.100.100",
        :mem => "4096",
        :cpu => "2"
    },
    {
        :name => "k8s-node02",
        :type => "node",
        :box => "centos/7",
        :box_version => "1902.01",
        :eth1 => "192.168.100.200",
        :mem => "4096",
        :cpu => "2"
    }
]

# This script installs Kubernetes via kubeadm after each box gets provisioned
$configBase = <<-SCRIPT

    # stop packages.cloud.google.com from transiting via
    sysctl -w net.ipv6.conf.all.disable_ipv6=1
    sysctl -w net.ipv6.conf.default.disable_ipv6=1
    echo "ip_resolve=4" >> /etc/yum.conf

    modprobe br_netfilter

    echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables

    sudo sysctl -p

    ## Install updates
    rpm --import https://download.docker.com/linux/centos/gpg
    rpm --import https://packages.cloud.google.com/yum/doc/yum-key.gpg
                               
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
    #   install prerequisites
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum -y update
    yum install -y yum-utils device-mapper-persistent-data lvm2 net-tools sshpass openssh-server install docker-ce docker-ce-cli containerd.io kubelet-1.22.3 kubeadm-1.22.3 kubectl-1.22.3 --disableexcludes=kubernetes

    # required for setting up passwordless ssh between guest VMs
    sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
    sudo service sshd restart   
    systemctl restart sshd.service

    # Set SELinux in permissive mode (effectively disabling it)
    setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

    #**
    # Switch Cgroupfs to be managed by systemd

    ## Create /etc/docker directory.
    mkdir /etc/docker

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
    "max-size": "100m"
    },
    "storage-driver": "overlay2",
    "storage-opts": [
    "overlay2.override_kernel_check=true"
    ]
}
EOF
    
    mkdir -p /etc/systemd/system/docker.service.d
    
    #** end switch to systemd

    # run docker commands as vagrant user (sudo not required)
    usermod -aG docker vagrant
    systemctl enable docker.service

    # Restart Docker
    systemctl daemon-reload
    systemctl restart docker
    
    # enable kubelet
    systemctl enable --now kubelet

    # kubelet requires swap off
    swapoff -a

    # keep swap off after reboot
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

    # ip of this box
    IP_ADDR=`ifconfig eth1 | grep Mask | awk '{print $2}'| cut -f2 -d:`
    set node-ip
    touch /etc/default/kubelet
    sudo sed -i "/^[^#]*KUBELET_EXTRA_ARGS=/c\KUBELET_EXTRA_ARGS=--node-ip=$IP_ADDR" /etc/default/kubelet
    sudo systemctl restart kubelet

SCRIPT

$Master = <<-SCRIPT

    echo "This is the master"

    modprobe br_netfilter
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables

    # pull k8s images
    kubeadm config --kubernetes-version=1.22.3 images pull

    # create an empty environment file
    sudo touch /etc/default/kubelet

    # ip of this box
    IP_ADDR=`ifconfig eth1 | grep Mask | awk '{print $2}'| cut -f2 -d:`

    # install k8s master
    HOST_NAME=$(hostname -s)
    kubeadm init --kubernetes-version=1.22.3 --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=172.16.0.0/16

    # copying credentials to regular user - vagrant
    sudo --user=vagrant mkdir -p /home/vagrant/.kube
    cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
    chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

    # install Calico pod network addon
    export KUBECONFIG=/etc/kubernetes/admin.conf
    
    # Install flannel CNI plugin
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

    # create a token for joining worker nodes
    kubeadm token create --print-join-command >> ./kubeadm_join_cmd.sh
    chmod +x ./kubeadm_join_cmd.sh

SCRIPT

$Node = <<-SCRIPT
    
    echo "This is a worker"

    modprobe br_netfilter
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
    echo 1 > /proc/sys/net/bridge/bridge-nf-call-ip6tables

    #configure kubectl
    sudo --user=vagrant mkdir -p /home/vagrant/.kube
    sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@k8s-master:/home/vagrant/.kube/config /home/vagrant/.kube/config
    sudo chown vagrant:vagrant /home/vagrant/.kube/config
    
    # join a worker node to the cluster
    sshpass -p "vagrant" scp -o StrictHostKeyChecking=no vagrant@k8s-master:~/kubeadm_join_cmd.sh .
    sudo sh ./kubeadm_join_cmd.sh
    
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.box_version = opts[:box_version]
            config.vm.hostname = opts[:name]

            config.vm.network "private_network", type: "bridge",
            dev: "virbr2",
            mode: "nat",
            network_name: "k8s", ip: opts[:eth1]

                config.vm.provider :libvirt do |domain|
                    domain.memory = opts[:mem]
                    domain.cpus = opts[:cpu]
                    domain.nested = true
                    domain.volume_cache = 'none'
                    domain.storage_pool_name = "vms"
                    
                end

            config.vm.provision "shell", inline: $configBase

            if opts[:type] == "master"
                config.vm.provision "shell", inline: $Master
            else
                config.vm.provision "shell", inline: $Node
            end

        end

    end

end
