---
layout: post
title:  "Accessing Private Repositories on GitHub Container Registry (GHCR) with k3s and RKE"
date:   2024-10-12 00:00:00
categories: Architecture
tags: featured
tipue_search_active: true
# image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.JPG
author:
  name: Pypycodes
  picture: /assets/images/author.jpg
tags: kubernetes  
---

## Accessing Private Repositories on GitHub Container Registry (GHCR) with k3s and RKE

In this guide, we'll walk through the steps to configure k3s and Rancher Kubernetes Engine (RKE) clusters to pull images from private repositories hosted on GitHub Container Registry (GHCR). By the end of this guide, your clusters will be able to securely access and deploy containers from private GHCR repositories.

## Prerequisites

- **k3s** and **RKE** clusters set up and configured.
- Administrator access to the clusters.
- A **Personal Access Token (PAT)** with the appropriate permissions on GitHub.

## Step 1: Create a Personal Access Token on GitHub

1. Go to [GitHub](https://github.com) and navigate to **Settings > Developer Settings > Personal Access Tokens > Tokens (classic)**.
2. Generate a new token with the following permissions:
   - `read:packages` (required to pull images)
   - `write:packages` (optional, if you intend to push images as well)
   - `repo` (if managing private repositories)
3. Copy the generated token as it will be required for authentication.

## Step 2: Create a Docker Config JSON File

To authenticate with GHCR, create a `.docker/config.json` file on your local machine with the following command:

```bash
echo '{"auths": {"ghcr.io": {"auth": "'$(echo -n USERNAME:PAT | base64)'"}}}' > ~/.docker/config.json
```

Replace `USERNAME` with your GitHub username and `PAT` with your personal access token.

## Step 3: Create a Kubernetes Secret for Image Pulling

Next, we'll create a Kubernetes secret that stores the Docker credentials. This will enable your k3s and RKE clusters to pull images from the private GHCR repository.

```bash
kubectl create secret generic ghcr-secret --from-file=.dockerconfigjson=~/.docker/config.json --type=kubernetes.io/dockerconfigjson -n default
```

This command creates a secret named `ghcr-secret` in the `default` namespace. You can replace the namespace with your specific namespace if required.

## Step 4: Configure k3s to Use the Secret

When deploying workloads in k3s that require images from private repositories, reference the `ghcr-secret` in your deployment YAML file, as shown below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-private-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-private-app
  template:
    metadata:
      labels:
        app: my-private-app
    spec:
      containers:
      - name: my-container
        image: ghcr.io/USERNAME/REPOSITORY:TAG
      imagePullSecrets:
      - name: ghcr-secret
```

Replace `USERNAME`, `REPOSITORY`, and `TAG` with the appropriate values for your image.

## Step 5: Configure RKE to Use the Secret

In RKE, you can either reference `imagePullSecrets` directly in workload definitions, or set up a default pull secret for the entire cluster. Here’s how to configure it in an RKE `cluster.yml` file:

```yaml
services:
  kube-api:
    secrets_encryption_config:
      enabled: true
kubelet:
  extra_binds:
    - "/var/run:/var/run"
addons: |
  ---
  apiVersion: v1
  kind: Secret
  metadata:
    name: ghcr-secret
    namespace: default
  type: kubernetes.io/dockerconfigjson
  data:
    .dockerconfigjson: $(cat ~/.docker/config.json | base64 -w 0)
```

### Referencing the Secret in Workloads

Similar to k3s, you can reference `imagePullSecrets` in your workload definitions when using RKE:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-private-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-private-app
  template:
    metadata:
      labels:
        app: my-private-app
    spec:
      containers:
      - name: my-container
        image: ghcr.io/USERNAME/REPOSITORY:TAG
      imagePullSecrets:
      - name: ghcr-secret
```

## Step 6: Verify the Setup

Deploy a test application using a private image from GHCR to verify the setup. If configured correctly, the pod should pull the image successfully and transition to a running state without authentication errors.

## Conclusion

By following these steps, you can securely access private repositories on GitHub Container Registry from both k3s and RKE clusters. This approach ensures that sensitive credentials are stored securely in Kubernetes secrets and only accessed by authorized workloads.

Happy deploying!
