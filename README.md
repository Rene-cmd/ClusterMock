Proxmox Cluster with High Availability and Kubernetes Deployment

This README provides a step-by-step guide to setting up a Proxmox cluster with High Availability (HA) and deploying Linux servers to run a Kubernetes cluster. This setup is ideal for educational purposes or as part of an exam project.
Table of Contents

    Prerequisites
    Cluster Setup
        Step 1: Install Proxmox VE
        Step 2: Configure Proxmox Cluster
        Step 3: Enable High Availability (HA)
    Kubernetes Setup
        Step 4: Deploy Linux VMs
        Step 5: Install Kubernetes
        Step 6: Configure Kubernetes Cluster
    Validation and Testing
    Troubleshooting
    Conclusion

Prerequisites

Before you begin, ensure you have the following:

    Proxmox VE Nodes: At least three physical servers or virtual machines with Proxmox VE installed.
    Network Configuration: A stable network configuration, with all Proxmox nodes able to communicate with each other.
    Root Access: SSH or physical access to each Proxmox node with root privileges.
    ISO Image: A Linux distribution ISO (e.g., Ubuntu Server) for creating VMs.
    Basic Knowledge: Familiarity with Proxmox, Linux command line, and Kubernetes concepts.

Cluster Setup
Step 1: Install Proxmox VE

    Download Proxmox VE:
        Visit the Proxmox Download Page and download the latest Proxmox VE ISO image.

    Install Proxmox VE:
        Boot each server from the Proxmox VE ISO and follow the installation prompts. Ensure that each server is installed with the same version of Proxmox VE.

    Initial Configuration:
        Configure network settings, hostname, and root password during installation.
        Once installed, access the Proxmox web interface via https://<server-ip>:8006.

Step 2: Configure Proxmox Cluster

    Create the Cluster:
        On the first node, run the following command to create the cluster:

        bash

    pvecm create exam-cluster

    Note down the join command provided after cluster creation.

Join Other Nodes:

    SSH into each additional Proxmox node and run the join command obtained in the previous step:

    bash

    pvecm add <ip-of-first-node>

Verify Cluster Status:

    On the primary node, verify that all nodes have joined the cluster:

    bash

        pvecm status

        You should see all nodes listed in the cluster.

Step 3: Enable High Availability (HA)

    Configure HA Manager:
        On the Proxmox web interface, navigate to Datacenter -> HA -> Resources.
        Select a VM or Container, click Add, and configure the HA settings.
        Repeat for each VM that requires HA.

    Test HA Configuration:
        Simulate a node failure by shutting down one node.
        The HA service should automatically migrate the affected VMs to another node.

Kubernetes Setup
Step 4: Deploy Linux VMs

    Create Linux VMs:
        In the Proxmox web interface, create three Linux VMs (e.g., Ubuntu Server). These VMs will serve as your Kubernetes master and worker nodes.
        Allocate appropriate CPU, RAM, and storage resources for each VM.

    Network Configuration:
        Ensure each VM is connected to the same network and can communicate with each other.

    Install SSH and Updates:
        SSH into each VM and install updates:

        bash

        sudo apt update && sudo apt upgrade -y
        sudo apt install -y ssh

Step 5: Install Kubernetes

    Install Docker:
        On each VM, install Docker as the container runtime:

        bash

    sudo apt install -y docker.io

Install Kubernetes Tools:

    Install kubeadm, kubelet, and kubectl:

    bash

        sudo apt-get install -y apt-transport-https ca-certificates curl
        curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
        sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl

Step 6: Configure Kubernetes Cluster

    Initialize Master Node:
        On the master node, initialize the Kubernetes cluster:

        bash

    sudo kubeadm init --pod-network-cidr=192.168.0.0/16

    Follow the output instructions to configure kubectl on the master node.

Join Worker Nodes:

    On each worker node, join the cluster using the command provided after initializing the master node:

    bash

    sudo kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash <hash>

Deploy a Pod Network:

    On the master node, deploy a pod network (e.g., Flannel):

    bash

        kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Validation and Testing

    Check Node Status:
        On the master node, verify that all nodes have joined the cluster and are in the Ready state:

        bash

    kubectl get nodes

Deploy a Test Application:

    Deploy a sample NGINX application to test the cluster:

    bash

        kubectl create deployment nginx --image=nginx
        kubectl expose deployment nginx --port=80 --type=NodePort

        Access the NGINX application through the NodePort from any browser.

    Test HA:
        Simulate a node failure and observe the Kubernetes cluster's behavior to ensure pods are rescheduled on available nodes.

Troubleshooting

    Proxmox Cluster Issues: If nodes are not joining the cluster, check network connectivity and firewall settings.
    Kubernetes Join Issues: If worker nodes fail to join the cluster, ensure that the token and discovery hash are correct and that the master node is reachable.
    HA Failover Issues: If HA does not work as expected, verify that HA is properly configured in the Proxmox web interface.

Conclusion

This guide has walked you through setting up a Proxmox cluster with High Availability and deploying Linux servers to run a Kubernetes cluster. This setup provides a robust and scalable environment suitable for testing and development purposes.
