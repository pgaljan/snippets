# About
Lightweight, frictionless, ephemeral Ubuntu-based K3S lab environment for quick tests, exploration of function, and training.  It does not require a cloud subscription and does not presume the use of Managed K8S.  I documented Virtualbox because it can be freely installed anywhere, and runs without elevation.  It is optional - any Ubuntu 2204 will work.\
\
A demonstrably useful setup should run fairly well on any machine with 8GB RAM, 4 cores, and 100GB SSD.

###### Requirements
  * [Ubuntu Server](https://ubuntu.com/download/server)
  * [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (optional)

Also helpful:  [K8S Cheatsheet](https://github.com/pgaljan/blog/blob/main/K8SCheat.md)\
Note: ubuntu will occasionally hang in vbox on boot with "*Begin: Loading Essential driver ...*".  Just reset the VM

##### Example VirtualBox Network setup

```mermaid
flowchart LR;
    Host---|Host-Only|Manager01;
    Manager01---LocalNetwork{{LocalNetwork}}
    LocalNetwork---Worker01
    LocalNetwork---Worker02
    LocalNetwork---Worker0n
    LocalNetwork---minio
    Host -.- LocalNetwork
    Internet(((Internet)))---LocalNetwork
```

     
<Details>
<summary>
Manager Node
</summary>

#### Virtualbox Config (optional)
2 vCPU, 2GB RAM, 25GB disk\
NIC 1: Bridged (will have an IP on the same as network your physical machine)\
NIC 2: Host-only (should have a 192.168.56.x IP - you ssh to this IP from the host)\
#### Prep the VM
```shell
sudo apt-get update
sudo apt upgrade -y
sudo apt install -y curl wget
```
#### Install K3S
*copy function may not work with this snippet*
```shell
curl -sfL https://get.k3s.io | sh -
sudo systemctl status k3s
```
#### Configure kubectl
```shell
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config && sudo chown $USER ~/.kube/config
sudo chmod 600 ~/.kube/config && export KUBECONFIG=~/.kube/config
```
#### Check K3S
```shell
sudo kubectl get nodes
sudo kubectl cluster-info
sudo kubectl get pods -A
```
#### Create a Deployment
```shell
sudo kubectl create deployment  nginx-deployment --image nginx --replicas 2
sudo kubectl get deployment nginx-deployment
sudo kubectl get pods
sudo kubectl expose deployment nginx-deployment --type NodePort --port 80
```
#### Verify the Deployment
```shell
ip a | grep "enp" | grep "inet " 
sudo kubectl get svc nginx-deployment
```
curl http://bridged_ip:mapped_port
#### Enable the dashboard
```shell
sudo kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
sudo kubectl get pods,svc -n kubernetes-dashboard
sudo kubectl patch svc kubernetes-dashboard --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]' -n kubernetes-dashboard
sudo kubectl get svc -n kubernetes-dashboard
```
#### Create dashboard yaml
```shell
vi dashboard.yaml
```
#### dashboard yaml contents
*copy function may not work with this snippet*
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```
#### Create service account. display token
```shell
sudo kubectl create -f dashboard.yaml
sudo kubectl -n kube-system  create token admin-user
```
#### Get IP and mapped port
```shell
ip a | grep "inet 192.168.56"
sudo kubectl get pods,svc -n kubernetes-dashboard
```
In a web browser on the host, go to https://192.168.56.x:3xxxx, paste token.
</Details>
<Details>
<summary>
Worker Node
</summary> 
#### Virtualbox Config (optional)
2 vCPU, 2GB RAM, 25GB disk\
NIC 1: Bridged (will have an IP on the same as network your physical machine)\
Note - second NIC is unnecessary - you will be able to ssh from the manager node if needed.
#### Prep the Node
```shell
sudo apt-get update
sudo apt upgrade -y
sudo apt install -y curl wget
```
#### Install K3S
*copy function may not work with this snippet*
```shell
curl -sfL https://get.k3s.io | sh –
sudo systemctl status k3s
```
#### Get Node token (on manager)
```shell
sudo cat /var/lib/rancher/k3s/server/node-token
```
#### Join the cluster
*copy function may not work with this snippet*
```shell
sudo curl -sfL https://get.k3s.io | K3S_URL=https://<bridged ip>:6443 K3S_TOKEN="<pasted_token>" sh -
```
#### Reboot
```shell
sudo reboot
```
#### Start the Agent
```shell
sudo systemctl enable --now k3s-agent
```
#### Verify the node join (on manager)
```shell
sudo kubectl get nodes
```
</Details>