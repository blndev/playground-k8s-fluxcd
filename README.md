# Repo: Playground-k8s-FluxCD.
This repo is an area to play with FluxCD https://fluxcd.io

FluxCD allows you to deploy kubernetes applications in a simple gitops manner.
It makes use of Helm-Charts, Kustomize and handles Git Repositories in an extreme simple way

This Documentation any my first steps are inspired by https://toolkit.fluxcd.io/get-started/

Kustomize is integrated into kubectl >= 1.4.
But to generate kustomize templates i suggest to install it on your system.

```
snap install kustomize
```
# DevEnv

FluxCD This run on any Kubernetes. You could use an AKS,EKS or whatever.
For my DevTest I use Rancher K3S on a Virtual Machine. 
To keep it more simple you could even use minikube.

# Setup FluxCD
1. Install Kubernetes 
   ````
   curl -sfL https://get.k3s.io | sh -
   ````
2. check your cluster: (``kubectl get pods``)
3. Install fluxcd 
   ```bash
   curl -s https://toolkit.fluxcd.io/install.sh | sudo bash
   ```

4. Create a Git Repo which should contain your Flux and Kubernetes Application Configuration (like this one :heart_eyes_cat:)

At this point you should have a Kubernetes Cluster and the flux binaries in the path. There is not flux pods yet in the cluster
## Configure FluxCD integration

First run a check to make sure all basics are working
```bash
flux check --pre
```
Note: If you using K3S you have to set the KUBECONFIG Path 

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml >> ~/.bashrc && source ~/.bashrc
```

### Using Bootstrap

Bootstrap is a function set integrated into fluxcd to simply activate the integration.
It is available for GIT Repositories hosted on Github or Gitlab. 
It  need a Git Access Token, because it will generate the repo if not existing and add / commit all the files required to manage the flux ecosystem.

**I will not recommend to do it this way because of the required Repo Write Access!**

See an alternate below.

Bootstrapping fluxCD:
```bash
flux bootstrap github \
  --owner=blndev \
  --repository=playground-k8s-fluxcd \
  --branch=main \
  --path=./clusters/demo-cluster \
  --personal
  

```

## Activate FluxCD without Bootstrap

This is an alternative and more flexible way. It does not need write access to the repo, can work with public repos without any key and support all other git Solutions, even self hosted.

I followed this guide if you need deeper Information: 
 https://toolkit.fluxcd.io/guides/installation/#generic-git-server

Note: All of the steps working also **without**(!) the flux cli. But then you have to write a lot of YAML by your own.
I heavily suggest to use flux here.

First clone the repo to any system where you have the flux cli installed.

```bash
git clone git@github.com:blndev/playground-k8s-fluxcd.git
cd playground-k8s-fluxcd

# prepare some structure
# this repo should also contain docs and more so i decided to follow the 
# best practice and created a clusters folder
# In addition i think about staging/production so a subfolder for the cluster as well
mkdir clusters/demo-cluster/flux-system

# This command will generate a big YAML file with all the flux components
flux install --version=latest \
  --export > ./clusters/demo-cluster/flux-system/gotk-components.yaml

# This command installs the custom resource definitions, operators etc.
# There is still no binding to the git repo.
# With this state you could work!!
kubectl apply -f ./clusters/my-cluster/flux-system/gotk-components.yaml
flux check

# Now we bind Flux to the Git (SSH and Authentication is also possible)
flux create source git flux-system \
  --url=https://github.com/blndev/playground-k8s-fluxcd.git \
  --branch=main \
  --interval=1m

# with this command we tell flux to observe the folder clusters-demo-cluster
# in the remote git repo for kustomization files
# from now it will take any of these and apply it to kubernetes
flux create kustomization flux-system \
  --source=flux-system \
  --path="./clusters/demo-cluster" \
  --prune=true \
  --interval=10m

# to be able to manage via gitops, we have to export the Source Config
# but still: this is not required and used only for infrasturcture management!!
flux export source git flux-system \
  > ./clusters/demo-cluster/flux-system/gotk-sync.yaml
# same applies to the kustomization which triggers the read of the source
flux export kustomization flux-system \
  >> ./clusters/demo-cluster/flux-system/gotk-sync.yaml

# now go online
git add .
git commit -a -m "activate flux"
git push origin main
```

The System is now online and actively watching for changes.
Changes could be build with flux or by hand.
# Install YOUR Application

We will install a simple Podinfo App (WebUI which shows info about the running pod)
As this repo is providing an kustomize file we can simple reference it.

If there is only a kubernets file on the path, the Kustomize controller will generate a kusomization - file.
See: https://github.com/fluxcd/kustomize-controller/blob/main/docs/spec/v1beta1/README.md

```bash
flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=30s \
  --export > ./clusters/demo-cluster/podinfo-source.yaml
  
 flux create kustomization podinfo \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --validation=client \
  --interval=5m \
  --export > ./clusters/demo-cluster/podinfo-kustomization.yaml
  ```
