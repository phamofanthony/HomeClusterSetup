# Overview
This document outlines installing and using the Kubernetes infrastructure on the nodes. The notes are largely based on the [official documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/).

# Node Preparation (All nodes)
### Disable swap
Required because Kubernetes needs precise memory control, and swap could break scheduling/eviction logic
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
### Install containerd
Our container runtime of choice that manages container lifecucles.
```bash
sudo apt update
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```
### Set SystemdCgroup to true in containerd's config
Makes containerd use the same systemd resource manager as the kubelet will, which will prevent cgroup mismatches that can cause pod startup failures. 

Systemd: Linux service and system manager that boots the OS, starts services, and manages system resources and processes

Cgroup (Control group): Linux kernel feature that limits, accounts, and isolates CPU, memory, and I/O usage for processes. K8s uses this to enforce resource limits on containers.
```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
```
### Enable IP forwarding on all nodes
```bash
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

# Installing kubeadm, kubelet, and kubectl
### Update packaeges and get dependencies for Kubernetes apt repository (All nodes)
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
### Get the public signing key for Kubernetes repositories
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
### Add the Kubernetes apt repository
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
### Update package index, install the main Kubernetes packages, and pin their versions
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
### Enable kubelet service 
```bash
sudo systemctl enable --now kubelet
```



# Control Plane Setup (Control plane node only)
### Initialize kubeadm w/ Calico CNI
```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=$(hostname -I | awk '{print $1}')
```
The output of this will include a command to run on worker nodes to join them. Save this. 
### Configure access with kubectl
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
After this, `kubectl get nodes` should show the control plane node as NotReady
### Install the Calico network plugin
Implements pod networking and network policy enforcement.
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```
Eventually, the control plane should turn to ready after this. You can check this by running `kubectl get pods -n kube-system`

# Worker Node Setup (Worker node only)
### Run the kubeadm join command from the earlier kubeadm output
```bash
sudo kubeadm join <CONTROL_PLANE_IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```
After a few minutes, run `kubectl get nodes`. You should see all nodes as Ready 
### Label nodes as worker
```bash
kubectl label node [NODE_NAME] node-role.kubernetes.io/worker=worker
```
