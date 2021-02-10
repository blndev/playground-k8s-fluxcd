# Playground-k8s-fluxcd.
Area to play with FluxCD


# DevEnv

This will run on any Kubernetes.
I use Rancher K3S on a Virtual Machine, but you could also use MiniKube or whatever else.

## Setup
1. Install Kubernets (e.g. minikube up or ```curl -sfL https://get.k3s.io | sh -````)
2. sudo snap install kustomize
3. check your cluster: kubectl get pods
4. Install fluxcd ```curl -s https://toolkit.fluxcd.io/install.sh | sudo bash ```
5. Create a Git Repo which should contain your Kubernetes Configuration (like this one ;)

# Configure FluxCD
I'm following this guide: https://toolkit.fluxcd.io/get-started/

flux check --pre

Note: If you using K3S you have to set the KUBECONFIG Path 
----
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
----

Then activate fluxCD
----
flux bootstrap github \
  --owner=blndev \
  --repository=playground-k8s-fluxcd \
  --branch=main \
  --path=./clusters/demo-cluster \
  --personal
  --git-readonly

----

Alternativ way (not github or gitlab): https://toolkit.fluxcd.io/guides/installation/#generic-git-server
git clone ...
cd repo ...
mkdir clusters/my-cluster/flux-system
flux install --version=latest \
  --export > ./clusters/my-cluster/flux-system/gotk-components.yaml

kubectl apply -f ./clusters/my-cluster/flux-system/gotk-components.yaml
flux check

flux create source git flux-system \
  --url=https://github.com/blndev/playground-k8s-fluxcd.git \
  --branch=main \
  --interval=1m

flux create kustomization flux-system \
  --source=flux-system \
  --path="./clusters/demo-cluster" \
  --prune=true \
  --interval=10m

flux export source git flux-system \
  > ./clusters/demo-cluster/flux-system/gotk-sync.yaml

flux export kustomization flux-system \
  >> ./clusters/demo-cluster/flux-system/gotk-sync.yaml



