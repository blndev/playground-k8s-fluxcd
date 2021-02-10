# Playground-k8s-fluxcd.
Area to play with FluxCD


# DevEnv

This will run on any Kubernetes.
I use Rancher K3S on a Virtual Machine, but you could also use MiniKube or whatever else.

## Setup
1. Install Kubernets (e.g. minikube up or ```curl -sfL https://get.k3s.io | sh -````)
2. check your cluster: kubectl get pods
3. Install fluxcd ```curl -s https://toolkit.fluxcd.io/install.sh | sudo bash ```
4. Create a Git Repo which should contain your Kubernetes Configuration (like this one ;)



