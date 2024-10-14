---
layout: post
title:  "Setting up K3S Kubernetes for Edge"
date:   2024-10-11 10:00:00
categories: Architecture
tags: featured
tipue_search_active: true
# image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.JPG
author:
  name: Pypycodes
  picture: /assets/images/author.jpg
tags: kubernetes  
---
Kubernetes, the leading platform for container orchestration, can sometimes be resource-intensive, especially for smaller deployments or edge computing environments. Enter **K3s** – a lightweight, certified Kubernetes distribution by Rancher Labs. It’s specifically designed to be low on resources, making it ideal for development, IoT, and edge use cases. 

In this guide, we'll walk through the process of setting up a K3s cluster. This article assumes you have basic knowledge of Linux commands and server setup.

## Prerequisites

Before starting, ensure you have:

- **Two or more Linux servers** (Ubuntu 20.04 or newer is preferred) with at least 1 GB of RAM and 1 vCPU each.
- **Sudo privileges** on each server.
- **Access to the internet** to download and install K3s.

## Step 1: Prepare the Environment

1. **Update the System Packages**  
   Log in to each server and update the package lists:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Disable Swap**  
   Kubernetes recommends disabling swap to ensure stable resource management:

   ```bash
   sudo swapoff -a
   ```

   To disable swap permanently, edit the `/etc/fstab` file and comment out any line that references swap.

3. **Install Necessary Tools**  
   On each server, install `curl` (if not already installed) and `iptables`:

   ```bash
   sudo apt install curl iptables -y
   ```

## Step 2: Install K3s on the Master Node

The master node, or server node in K3s terminology, will manage the cluster and handle control plane activities.

1. **Run the K3s Installation Script**  
   On your primary server, execute the following command to install K3s:

   ```bash
   curl -sfL https://get.k3s.io | sh -
   ```

   This script will download and install K3s. It also sets up systemd services to automatically start K3s on boot.

2. **Verify the Installation**  
   Check the status of K3s to ensure it’s running:

   ```bash
   sudo systemctl status k3s
   ```

   You should see the K3s service listed as active.

3. **Retrieve the Node Token**  
   To join worker nodes, you’ll need the **node token**, which is generated during the K3s installation. Retrieve it with:

   ```bash
   sudo cat /var/lib/rancher/k3s/server/node-token
   ```

   Copy the token as you’ll need it in the next step to connect worker nodes to the master node.

## Step 3: Install K3s on Worker Nodes

On each worker node, you will install K3s with a command that points to the master node’s IP and uses the node token.

1. **Run the K3s Installation Script with Master Node Details**  
   Replace `<MASTER_IP>` with the IP address of your master node and `<NODE_TOKEN>` with the token you copied in the previous step:

   ```bash
   curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -
   ```

2. **Verify Node Connection**  
   After a minute or so, check the status of K3s on each worker node:

   ```bash
   sudo systemctl status k3s-agent
   ```

3. **Confirm Nodes Are Connected**  
   On the master node, list the nodes to verify they’re successfully connected:

   ```bash
   sudo k3s kubectl get nodes
   ```

   You should see both the master and worker nodes listed.

## Step 4: Testing the Cluster

To ensure everything is functioning correctly, let’s deploy a simple application.

1. **Deploy an Nginx Pod**  
   Use the following command on the master node to deploy an Nginx pod:

   ```bash
   sudo k3s kubectl create deployment nginx --image=nginx
   ```

2. **Verify Pod Status**  
   Check the status of the newly created Nginx pod:

   ```bash
   sudo k3s kubectl get pods
   ```

3. **Expose Nginx Deployment as a Service**  
   Make the Nginx deployment accessible by exposing it as a LoadBalancer service:

   ```bash
   sudo k3s kubectl expose deployment nginx --type=LoadBalancer --port=80
   ```

4. **Retrieve the Service IP**  
   List the services to get the external IP:

   ```bash
   sudo k3s kubectl get services
   ```

   Access this IP from your browser to see the Nginx welcome page, verifying your K3s setup is successful.

## Step 5: Configuring kubectl Access Locally

To manage the K3s cluster from your local machine, you can copy the kubeconfig file.

1. **Copy the kubeconfig File**  
   Run the following command on the master node to copy the configuration file:

   ```bash
   sudo cat /etc/rancher/k3s/k3s.yaml
   ```

   Copy the output and save it as `k3s.yaml` on your local machine.

2. **Configure kubectl to Use the K3s Config File**  
   On your local machine, set the `KUBECONFIG` environment variable:

   ```bash
   export KUBECONFIG=/path/to/k3s.yaml
   ```

3. **Verify kubectl Connectivity**  
   Check the nodes from your local machine to verify the connection:

   ```bash
   kubectl get nodes
   ```

   If you see the list of nodes, you’ve successfully connected to the K3s cluster!

## Conclusion

Congratulations! You have successfully set up a lightweight Kubernetes cluster using K3s. This setup is perfect for development environments, small-scale deployments, and edge computing applications. K3s is a versatile and resource-efficient way to run Kubernetes, bringing the power of container orchestration to resource-constrained environments.

Feel free to explore further by deploying additional workloads, experimenting with Helm charts, and managing your cluster with various Kubernetes tools. Happy coding!
