# recipe for installing:
- fish shell 
- ansible
- k3s
- helm
- minecraft server
- ingress-nginx

on fresh Ubuntu 20.04.2 LTS 32 bit 
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

vi inventory/hosts.ini

vi inventory/groups_vars/all.yml

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

