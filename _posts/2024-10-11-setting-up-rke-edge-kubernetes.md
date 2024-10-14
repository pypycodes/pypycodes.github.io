---
layout: post
title:  "How to setup Kubernetes using Rancher Kubernetes Engine"
date:   2024-10-11 10:00:00
categories: [RKE, Kubernetes for Edge,Architecture]
tags:
  - featured
  - kubernetes
  - k8s
read_time: true
excerpt: Rancher Kubernetes Engine (RKE) is a Kubernetes installer that simplifies the deployment of Kubernetes clusters in various environments. RKE is known for its ease of use and compatibility with multiple platforms, including on-premises servers and cloud environments. This guide will take you through setting up a Kubernetes cluster with RKE on Linux servers.
---
Rancher Kubernetes Engine (RKE) is a Kubernetes installer that simplifies the deployment of Kubernetes clusters in various environments. RKE is known for its ease of use and compatibility with multiple platforms, including on-premises servers and cloud environments. This guide will take you through setting up a Kubernetes cluster with RKE on Linux servers.

## Prerequisites

Before you begin, ensure you have:

- **Three or more Linux servers** (Ubuntu 20.04 or newer is preferred) with at least 2 GB of RAM and 2 vCPUs each.
- **Sudo access** on each server.
- **Access to the internet** for downloading necessary software.
- **Docker** installed on each server. If Docker is not installed, follow the instructions in Step 1.

## Step 1: Prepare the Servers

### 1.1 Install Docker

If Docker is not already installed, use the following commands to set it up on each server:

```bash
curl -fsSL https://get.docker.com | sh
sudo systemctl enable docker && sudo systemctl start docker
```

Verify Docker installation:

```bash
docker --version
```

### 1.2 Disable Swap

Kubernetes requires swap to be disabled for stable operation. Run the following command to temporarily disable swap:

```bash
sudo swapoff -a
```

To disable swap permanently, open the `/etc/fstab` file and comment out any swap entries.

### 1.3 Configure Firewall (Optional)

If you have a firewall enabled, make sure to allow traffic on the necessary ports for Kubernetes:

```bash
# Allow traffic on port 6443 for Kubernetes API
sudo ufw allow 6443

# Allow traffic for communication between nodes (ports 2379, 2380, 10250, 10255, etc.)
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250:10255/tcp
```

## Step 2: Install RKE

1. **Download the RKE Binary**  
   On your local machine, download the latest RKE binary:

   ```bash
   wget https://github.com/rancher/rke/releases/download/v1.3.13/rke_linux-amd64 -O rke
   ```

2. **Make the RKE Binary Executable**  
   Change the permissions to make the file executable:

   ```bash
   chmod +x rke
   ```

3. **Move RKE to a System Path**  
   To access RKE from anywhere, move it to `/usr/local/bin`:

   ```bash
   sudo mv rke /usr/local/bin/
   ```

4. **Verify Installation**  
   Confirm the installation by checking the RKE version:

   ```bash
   rke --version
   ```

## Step 3: Configure the Cluster

### 3.1 Create a Cluster Configuration File

1. **Generate the Cluster Configuration**  
   On your local machine, run the following command to generate a cluster configuration file (`cluster.yml`):

   ```bash
   rke config --name cluster.yml
   ```

   Follow the prompts to specify the details of your cluster. You will need to provide the IP addresses of your nodes, set roles (etcd, controlplane, worker), and optionally define SSH keys or passwords for accessing each node.

2. **Edit the `cluster.yml` Configuration**  
   Once the file is generated, you can manually edit the `cluster.yml` file to make any adjustments. The file should include at least one etcd node, one control plane node, and one or more worker nodes.

   Here’s an example configuration for a simple three-node cluster:

   ```yaml
   nodes:
     - address: 192.168.1.101
       user: ubuntu
       role: [etcd, controlplane, worker]
       ssh_key_path: ~/.ssh/id_rsa

     - address: 192.168.1.102
       user: ubuntu
       role: [worker]
       ssh_key_path: ~/.ssh/id_rsa

     - address: 192.168.1.103
       user: ubuntu
       role: [worker]
       ssh_key_path: ~/.ssh/id_rsa
   ```

### 3.2 Validate Node Access

Ensure that you can SSH into each node without needing to enter a password, as RKE relies on SSH for deployment. Test SSH access with the following command:

```bash
ssh ubuntu@192.168.1.101
```

## Step 4: Deploy the Cluster

1. **Run the RKE Deployment**  
   With the `cluster.yml` file configured, you can deploy your cluster:

   ```bash
   rke up --config cluster.yml
   ```

   RKE will connect to each node, deploy Kubernetes components, and set up the cluster based on your configuration. The process may take several minutes.

2. **Check for Errors**  
   Monitor the output for any errors. If RKE completes successfully, you will see messages indicating that the cluster has been successfully deployed.

## Step 5: Access the Cluster

### 5.1 Retrieve kubeconfig File

After deployment, RKE generates a `kube_config_cluster.yml` file in the same directory. This file is required to access and manage the cluster with `kubectl`.

1. **Move kubeconfig to .kube Directory**  
   Copy the generated kubeconfig file to your `.kube` directory:

   ```bash
   mkdir -p ~/.kube
   cp kube_config_cluster.yml ~/.kube/config
   ```

2. **Verify Cluster Access**  
   Check the cluster status with `kubectl`:

   ```bash
   kubectl get nodes
   ```

   You should see a list of all nodes in the cluster, showing their roles and statuses.

## Step 6: Testing the Cluster

To ensure the cluster is functioning correctly, let’s deploy a simple application.

1. **Deploy an Nginx Deployment**  
   Run the following command to create an Nginx deployment:

   ```bash
   kubectl create deployment nginx --image=nginx
   ```

2. **Expose the Nginx Deployment**  
   Expose the deployment as a LoadBalancer service:

   ```bash
   kubectl expose deployment nginx --type=LoadBalancer --port=80
   ```

3. **Retrieve the Service Information**  
   Check the service details and obtain the external IP:

   ```bash
   kubectl get services
   ```

   You can now access Nginx by navigating to the external IP in a browser.

## Conclusion

You have successfully set up a Kubernetes cluster using Rancher Kubernetes Engine! This setup provides a robust, production-grade Kubernetes environment that you can use for development, testing, and deployment. Explore further by deploying different workloads and integrating tools to manage your cluster.

RKE’s flexibility and simplicity make it a popular choice for deploying Kubernetes clusters across diverse environments. With your new cluster, you’re ready to dive into Kubernetes and start deploying containerized applications.

This article should provide a comprehensive and clear overview of setting up an RKE cluster, suitable for anyone looking to get started with Kubernetes in a manageable way. Enjoy working with your RKE-powered Kubernetes cluster!
