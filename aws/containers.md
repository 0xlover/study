# Kubernetes(K8)
TO BE SORTED
Orchestrate Docker containers in clusters for automatic deployment and scaling
## Components
The Control Pane orchestrates containered applications(like Docker images running on Kubernetes Nodes) 
Cluster Nodes are virtual or physical servers in a Kubernetes cluster
Nodes contain a Container Runtime, a Kubelet(used to intereact with the Kubernetes API running on the Control Pane) and Pods
Pods are the smallest unit in Kubernetes handling containers, a pod may and often have only one container
A minimal setup would have the Contol Pane and a single Node with the Container Runtime and a Kubelet running maybe a Pod
These components have names!
* Kubernets API is known as the "kube-apiserver"
* There's a simple key-value database store called "etdb" running on the cluster to handle it
* The Scheduler, "kube-scheduler" constantly checks for pods with a missing node and assigns one based on requirements
* On every node a "kube-proxy" is running
* To interact with more then one cloud provider there's the optional "cloud-contoller-manager" **What platforms are supported?**
Documentation https://kubernetes.io/docs/concepts/overview/kubernetes-api
Controllers for nodes, jobs, endpoints and accounts are available
## Support
Podman by Redhat is supported by Kubenertes
## Todo
Expand on Docker, Podman, actually try deploying on EKS(Elastic Kubernetes Service)