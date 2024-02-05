# Automate local deployment of stefanprodan/podinfo via Vagrant

Podinfo is a tiny web application made with Go that showcases best practices of running microservices in Kubernetes. More on that [here](https://github.com/stefanprodan/podinfo).This Vagrantfile is designed to provision a VirtualBox virtual machine with an Ubuntu 23.04 base box, configuring it as a node for a Kubernetes cluster. The setup includes the installation of essential dependencies, Kubernetes (using kind), Helm, NGINX, and the deployment of the podinfo application.

# Prerequisites
Before using this Vagrantfile, ensure that you have the following prerequisites installed on your host machine:
* VirtualBox
* Vagrant


## Usage
1. Clone the repository:
```bash
git clone <repository-url>
cd <repository-directory>
```
2. To create the Ubuntu image with the Kubernetes cluster and the application running on it:
```
vagrant up 
```
3. To SSH into the Ubuntu machine
```
vagrant ssh
```
4. To cleanup the Ubuntu machine
```
vagrant destroy 
```

# Configuration
## VM Configuration
* Memory: 2048 MB
* CPUs: 2
* Network: Private network with static IP 192.168.99.101

## Provisioning Script
The provisioning script ($common) performs the following tasks:

1. Updates package repositories (sudo apt-get update).
2. Installs kind, Docker, kubectl, Helm, and NGINX.
3. Creates a Kubernetes cluster using the configuration in clusterconfig.yaml.
4. Deploys the podinfo application using Helm.
5. Applies a NodePort service (svc-nodeport.yaml) for external access.
6. Installs NGINX, configures SSL certificates, and sets up a reverse proxy for secure access.

## List of Kubernetes resources deployed in the "test" namespace
![automate-podinfo](https://github.com/S7aso1/vagrant-podinfo-cluster/assets/54038482/6f604855-ebff-440b-ad4b-1645d262c799)

## Diagram & short explanation
1. Reaching the app from Inside the Cluster:
* In your Kubernetes cluster, the frontend-service is exposed as a NodePort service.
* The NodePort service listens on port 80 inside the cluster and forwards traffic to port 9898 on the pods with the specified labels.
* So, within the cluster network, you can reach the frontend application by accessing it on port 80.

2. Outside the Cluster (For example a machine in the Same Network as the Ubuntu Machine):
* Since you have an additional port mapping on the control plane node in your clusterconfig.yaml file:
```
extraPortMappings:
  - containerPort: 30000
    hostPort: 8080
    protocol: TCP
```
* This means that if you are outside the cluster, you access the frontend application using port 8080 on the IP address of the control plane node.
* For example, if your control plane node's IP address is 192.168.99.101, you would access the application from your laptop using: `http://192.168.99.101:8080`
* This request is then forwarded to the NodePort service on port 30000, which, in turn, routes the traffic to the frontend application on port 9898.
### These examples are shown in the diagram below:
![Untitled Diagram drawio](https://github.com/S7aso1/vagrant-podinfo-cluster/assets/54038482/08e580fa-6efb-4fc6-9104-cdd441c23b17)
