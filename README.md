k8s lab with gitops via ArgoCD
==============================
k8s cluster (1 master + X nodes )

k8s ver 1.21.0

## Info

This lab is designed for local testing different k8s features and deployment with ArgoCD.
All you need is: Vargant, VirtualBox and some time to deploy it.

The K8s cluster in its basic configuration consists of two virtual machines: master and node, if you want more nodes you can change **default_node_count** var, or create a lab with the `K8S_NODE_COUNT=X vagrant up` command.

Once the cluster is installed and configured, Argo comes in and installs all the applications from the argo/app directory for the current repository.

![localk8s](https://raw.githubusercontent.com/fl64/localk8s/dev3/scheme/localk8s.png)

## Requirements:
- virtualbox (tested on 6.1.12)
- vagrant (tested on 2.2.14)
- ansible (tested on 4.2.0)

```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.kubernetes
```

## Recommendations
- envrc
- k9s
## Current setup

- k8s
- ArgoCD + ArgoCD applicationset
- Metallb
- Nginx ingress
- Prometheus
- Grafana
- nfs-client-provisioner (storageClass: nfs-client)
- vactor-agent

## How to

```bash
# up VMs and run ansible playbook
vagrant up

# run playbooks for specified ansible tags
K8S_TAGS=common vagrant up

# ssh to VMs
vagrant ssh master
vagrant ssh node100
etc...

# how to get to UI with `k port-forward`
## how to get into the Argo UI
k port-forward service/argocd-server -n argocd 8080:80
browse to http://localhost:8080

## how to get into the Prometheus UI
k port-forward service/prometheus-server -n prometheus 9090:80
browse to http://localhost:9090

## how to get into the Prometheus UI
k port-forward service/grafana -n grafana 3000:80
browse to http://localhost:3000

# Another way with ingress and /etc/hosts
## Add local records to /etc/hosts
sudo bash -c "cp /etc/hosts /etc/hosts.backup && export HOSTS_PATCH=\"$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath=\"{.status.loadBalancer.ingress[0].ip}\") grafana.k8s.local argo.k8s.local prom.k8s.local\"; grep -qF \"${HOSTS_PATCH}\" -- /etc/hosts || echo \"${HOSTS_PATCH}\" >> /etc/hosts"

## Browse
https://grafana.k8s.local
https://argo.k8s.local
https://prom.k8s.local

## default login and pass everywhere: admin/password

# clean up
vagrant down
```

## Some useful stuff

### Bash completions:
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
### Vim settings:
```bash
cat <<EOF >> ~/.vimrc
set number
set et
set sw=2 ts=2 sts=2
EOF
```
### krew plugin manager
- install krew: https://krew.sigs.k8s.io/docs/user-guide/setup/install/
- plugins list: https://krew.sigs.k8s.io/plugins/
plugins

```
kubectl krew install access-matrix
kubectl krew install view-utilization
kubectl krew install view-webhook
kubectl krew install example
```
