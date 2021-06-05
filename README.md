My local CNCF CKA preparation lab
=================================
k8s cluster (1 master + X nodes )

## Info

This lab is designed for local testing different k8s features and deployment with ArgoCD.
All you need is: Vargant, VirtualBox and some time to deploy it.

The K8s cluster in its basic configuration consists of two virtual machines: master and node, if you want more nodes you can change **default_node_count** var, or create a lab with the `K8S_NODE_COUNT=X vagrant up` command.

Once the cluster is configured, Argo comes in and installs all the applications from the argo/app directory for the current repository.

![localk8s](https://raw.githubusercontent.com/fl64/localk8s/dev3/scheme/localk8s.png)

## Requirements:
- virtualbox (tested on 6.1.12)
- vagrant (tested on 2.2.14)
- envrc (options)
## Current setup

- k8s
- ArgoCD + ArgoCD applicationset
- Metallb
- Nginx ingress
- Prometheus

## How to

``` bash
# up VMs and run ansible playbook
vagrant up

# run playbooks with git tag
K8S_TAGS=common vagrant up

# ssh to VMs
vagrant ssh master
vagrant ssh node100
etc...

# clean up
vagrant down
```

## Prep:
(Already embedded in VM OS)

Add some completions:
```bash
cat <<EOF >> ~/.bashrc
source <(kubectl completion bash)
source <(kubeadm completion bash)
alias k=kubectl
complete -F __start_kubectl k
export do="--dry-run=client -oyaml"
EOF

source ~/.bashrc
```
Useful vim settings:
```bash
cat <<EOF >> ~/.vimrc
set number
set et
set sw=2 ts=2 sts=2
EOF
```

Links:
- https://github.com/argoproj/argocd-example-apps/tree/master/helm-dependency
