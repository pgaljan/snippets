# About
Super lightweight, frictionless, ephemeral K3S lab environment for quick tests, exploration of function and training.  I have documented Virtualbox because it can be freely installed anywhere, runs without elevation, doesn't need a cloud or software subscription, etc.  I believe NAT could be used, but I leverage bridged networks for my lab.  If using virtualbox, it's necessary to create a host-only adapter on any manager node from which you'd like to access the kubernetes dashboard.  Host-only adapters are not necessary for worker nodes.
<Details>
<summary>
Requirements
</summary>

  * [Ubuntu Server](https://ubuntu.com/download/server)
  * [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (optional)
</Details>

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
```shell
curl -sfL https://get.k3s.io | sh –
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
ip a | grep "192.168.56"
sudo kubectl get svc nginx-deployment
curl <ip>:<mapped port>
```
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
```yaml
apiVersion: v1apiVersion: v1
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
sudo kubectl create -f k3s-dashboard.yaml
sudo kubectl -n kube-system  create token admin-user
```
#### Get IP and mapped port
```shell
ip a | grep "inet 192.168.56"
sudo kubectl get pods,svc -n kubernetes-dashboard
```
</Details>
<Details>
<summary>
Worker Node
</summary>

#### Virtualbox Config (optional)
2 vCPU, 2GB RAM, 25GB disk\
NIC 1: Bridged (will have an IP on the same as network your physical machine)\
Note - second NIC is unnecessary - you will be able to ssh from the manager node if needed.\

#### Prep the Node
```shell
sudo apt-get update
sudo apt upgrade -y
sudo apt install -y curl wget
```
#### Install K3S
```shell
curl -sfL https://get.k3s.io | sh –
sudo systemctl status k3s
```
#### Get Node token (on manager)
```shell
sudo cat /var/lib/rancher/k3s/server/node-token
```
#### Join the cluster
```shell
sudo curl -sfL https://get.k3s.io | K3S_URL=https://<bridged ip>:6443 K3S_TOKEN="<pasted_token>" sh -
```
#### Start the Agent
```shell
sudo systemctl enable --now k3s-agent
```
#### Verify the node join
```shell
sudo kubectl get nodes
```
</Details>
<Details>
<summary>
Frequent Operational Commands
</summary>

##### Get mapped port for the dashbaord
```shell
sudo kubectl get pods,svc -n kubernetes-dashboard
```
##### Get the dashboard token
```shell
sudo kubectl -n kube-system  create token admin-user
```
##### Get Virtualbox Host-only IP address for ssh & dashboard
```shell
ip a | grep "inet 192.168.56"
```

##### K3S Status
```shell
sudo systemctl status k3s
```
##### Quick Inventory
```shell
clear
sudo kubectl get node -o wide
read -n 1 -r -s -p $'Showing Nodes. Press enter to continue...\n'
clear
sudo kubectl get pods --all-namespaces -o wide
read -n 1 -r -s -p $'Showing Pods. Press enter to continue...\n'
clear
sudo kubectl get deployment --all-namespaces -o wide
read -n 1 -r -s -p $'Showing Deployment. Press enter to continue...\n'
clear
sudo kubectl get service --all-namespaces -o wide
read -n 1 -r -s -p $'Showing Service. Press enter to continue...\n'
clear
sudo kubectl get rs --all-namespaces -o wide
read -n 1 -r -s -p $'Showing Service. Press enter to continue...\n';clear
```
</Details>
