---
title: Installing Kubernetes Cluster
date: 2023-12-03 01:28:45S +0530
categories: [Kubernetes]
tags: [kubernetes]
author: Raghavendra
toc: true
![kubernetes](/images/pics%20(1).png){: width="450" height="236" }---

**Step-1 Create 3 VMs on a platform of your choice: Azure, AWS or GCP**

-   Names: master, node1, node2
-   OS: Ubuntu 22.04 LTS

**Step-2 Execute the following on all the 3 nodes**

Note: Make sure you adjust your Private IPs.

    # Update the VMs
    sudo apt update
    sudo apt-upgrade -y
    sudo apt-get remove needrestart -y
    
    # Add entries in hosts file
    sudo nano /etc/hosts
    192.168.1.4 master
    192.168.1.5 node1
    192.168.1.6 node2
    
    # Turn off swap (Most cloud VMs don't have swap turned on by default, but still)
    sudo swapoff -a
    
    # Forwarding IPv4 and letting iptables see bridged traffic
    sudo tee /etc/modules-load.d/containerd.conf <<EOF
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    
    # sysctl params required by setup, params should persist across reboots
    # Sysctl (system control) variables control certain kernel parameters that influence the behavior of different parts of the operating system
    sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF
    
    # Apply sysctl params without reboot
    sudo sysctl --system
    
    # OPTIONAL
    # Verify that the br_netfilter, overlay modules are loaded by running the following commands:
    lsmod | grep br_netfilter
    lsmod | grep overlay
    
    # Install a few utils
    sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
    
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
    
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable" -y
    
    # Update the system
    sudo apt update
    
    # Install container runtime
    sudo apt install -y containerd.io
    
    containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
    
    sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    
    # Install kubernetes
    curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
    
    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list -y
    
    sudo apt update
    sudo apt install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

