# recipe for installing:
- fish shell 
- ansible
- k3s
- helm
- minecraft server
- ingress-nginx
- argo-cd

on fresh Ubuntu 20.04.2 LTS 64 bit 
on minimum 2 Raspberry Pi 4B

note: commands below are for local Ubuntu environment (not k3s master or worker nodes)

## fish shell
```
sudo apt-get install fish

fish
```

## ansible

```
sudo apt-get install ansible -y
```

## k3s

```
git clone https://github.com/rancher/k3s-ansible

cd k3s-ansible

vi inventory/sample/hosts.ini

vi inventory/sample/groups_vars/all.yml

ansible-playbook site.yml -i inventory/hosts.ini 
```

## helm

```
wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz

tar -zxvf helm-v3.5.4-linux-amd64.tar.gz

mv linux-amd64/helm /usr/local/bin/helm
```

## minecraft

```
helm repo add itzg https://itzg.github.io/minecraft-server-charts/

helm repo update

kubectl create namespace minecraft

helm install --namespace minecraft minecraft -f values.yaml itzg/minecraft

```

## ingress-nginx

```
kubectl delete -n kube-system all,secrets,pvc -l release=traefik

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --set tcp.25565="minecraft/minecraft-minecraft:25565"
```

## argo-cd
To install argocd with ARM support, a fork from alinbalutoiu is used.
It might be necessary to label node that argocd should be installed on,
since by default it installs on master node.

```
kubectl label nodes <worker-node> kubernetes.io/partition=argocd

helm repo add argo https://argoproj.github.io/argo-helm

helm install my-argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace \
    --set global.image.repository="alinbalutoiu/argocd" \
    --set installCRDs="false" \
    --set nodeSelector.kubernetes.io/partition=argocd \
    --wait

kubectl port-forward service/my-argocd-server -n argocd 8080:443
```
Now argo-cd ui should be accessible from browser via https://localhost:8080
By default initial credentials are username:admin password:<name of argocd-server pod>

```
kubectl edit secret argocd-secret -n argocd
```
4. Restart argocd-server pod
```
kubectl delete pod <argocd-server-pod_name> -n argocd
```

To deploy minecraft via argocd-ui
1. Add repository url via Repositories -> Connect repo using HTTPS
   https://github.com/itzg/minecraft-server-charts/

2. New app and then fill out configuration settings. 
   - imageTag: multiarch
   - minecraftServer.eula: true
   - minecraftServer.type: PAPER
   - persistence.dataDir.enabled
   - resources.requests.cpu: 2000m
   - resources.requests.memory: 2048Mi

