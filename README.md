My local CNCF CKA preparation lab
=================================
k8s cluster (1 master + X nodes )

## Requirements:
- virtualbox (tested on 6.1.12)
- vagrant (tested on 2.2.14)

## Current setup

- k8s
- ArgoCD + ArgoCD applicationset
- Metallb
- Nginx ingress


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
## Enable metrics service

https://github.com/kubernetes-sigs/metrics-server

added args for metrics-server container to https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml:
```yaml
      - --v=2
      - --kubelet-insecure-tls
      - --kubelet-preferred-address-types=InternalIP
```
than
```bash
kubectl apply -f /deploy/04_metrics_server
```

## Usefil links

### Resources used in lab design:
- Ansible & k8s:
  - https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-16-04-ru
- Metallb:
  - https://metallb.universe.tf/
  - https://kubernetes.github.io/ingress-nginx/deploy/baremetal/
- NFS
  - https://medium.com/@myte/kubernetes-nfs-and-dynamic-nfs-provisioning-97e2afb8b4a9?_branch_match_id=767317559398297931
  - https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client
- nginx-ingress
  - Nginx-ingress: kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml
- LABS:
  - https://github.com/knrt10/kubernetes-basicLearning
