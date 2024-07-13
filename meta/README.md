# Overview

This Ansible playbook is designed to automate the generation of configuration files and resource manifests to install a [Meta (Facebook Messenger and Instagram) bridge](https://docs.mau.fi/bridges/go/meta/index.html) for Matrix using Mautrix. 

The base configuration is set for Messenger (the bridge example config is set to Instagram).

It uses sqlite database on a PersistentVolumeClaim.

# Note

The playbook is by intention not a deployment playbook, it just helps you generating consistent configuration files and manifests.

# Components
**Playbook**: generate_meta_bridge_resources.yaml

This playbook runs tasks to generate configuration and Kubernetes manifest files required for deploying the Meta bridge.
It uses Jinja2 templates to customize the files based on user-provided variables.

**Variables file**: vars/main.yaml.example

This file contains the variables needed to customize the templates. Users should copy this file to vars/main.yaml and fill in the appropriate values.

**Templates**

meta_app_service.yaml: Defines the application service for the Meta bridge. This is what you need to install on your Matrix server.

config.yaml: Contains the configuration for the Mautrix Meta bridge. Will be added as a ConfigMap on the kubernetes cluster.

k8s.yaml: Kubernetes manifests for deploying the Mautrix Meta bridge.

# How to Use

## Install Ansible

Ensure that Ansible is installed on your system. You can install Ansible using the following command:

```bash
sudo apt-get update
sudo apt-get install ansible
```

or


```bash
pip install ansible
```

## Prepare Variables File

Copy the example variables file to the vars directory:

```bash
cp vars/main.yaml.example vars/main.yaml
```

Edit the vars/main.yaml file to provide the necessary values:

```yaml
matrix_domain: matrix.example.com
admin_user: admin
appservice_host_or_ip: 10.0.0.2
appservice_port: 29319
as_token: YOUR_AS_TOKEN
hs_token: YOUR_HS_TOKEN
storage_class_name: YOUR_K8S_STORAGE_CLASS
```

## Run the Playbook

Execute the playbook to generate the configuration and Kubernetes manifest files:

```
ansible-playbook  -i localhost -e ansible_python_interpreter=`which python`  generate_meta_bridge_resources.yaml
```

## Deploy to Kubernetes

After running the playbook, the generated files will be located in /tmp:

- /tmp/mautrix--meta-config.yaml
- /tmp/mautrix--meta-k8s.yaml
- /tmp/mautrix--meta_app_service.yaml

Create the ConfigMap:

```bash
kubectl create configmap mautrix-meta  --from-file /tmp/mautrix--meta-config.yaml
```


Apply the Kubernetes manifests:

```bash
kubectl apply -f /tmp/mautrix--meta-k8s.yaml
```

## Configure your Matrix server

Register the new app service on your Matrix server using `/tmp/mautrix--meta_app_service.yaml`

## Network Connectivity

I defined the app service as NodePort as I'm running it on a very simple, minimal, single node kubernetes "cluster".  In my case the matrix server is running on a VM and the Kubernetes server with the bridge service on it on a separate VM, both connected with ZeroTier. I specified `localhost` as default application service host. I hardly believe this would work at all, maybe I should have assumed that the matrix server is running on the same k8s cluster and it could use the cluster internal service DNS... 🤔 Whatever... your mileage might vary.



