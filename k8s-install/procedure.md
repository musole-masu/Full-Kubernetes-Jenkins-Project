# Setting Up A Simple Kubernetes Cluster with Kubeadm (v1.29)
**Outcomes**: A production-ready Kubernetes cluster with a 3-node setup: 1 master node and 2 worker nodes. 

**Prerequisites**: 3 virtual machines running Ubuntu 20.04 LTS (or the latest release), each with a minimum of 2 CPUs, 4 GB of RAM, and 50 GB of storage.
## Nodes Configuration (Master and Worker)

### 1. Disable SWAP Memory

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

```

### 2. Install Docker

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl restart docker
sudo systemctl status docker

```
### 3. Forwarding IPv4 and letting iptables see bridged traffic

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

```

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

```

```bash
sudo sysctl --system

```

```bash
lsmod | grep br_netfilter
lsmod | grep overlay

```

```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

```

### 4. Configure Network Ports and Protocols for Kubernetes on AWS or On-Premise.

![Kubernetes Cluster - Control Plane Node Port and Protocol](https://res.cloudinary.com/dnw22sl1p/image/upload/v1733092417/Full-K8s-Jenkins-Project/Control_Plane_Port_3_PM_tglb73.png)

![Kubernetes Cluster - Worker Node Port and Protocol](https://res.cloudinary.com/dnw22sl1p/image/upload/v1733092417/Full-K8s-Jenkins-Project/Worker_node_port_PM_vlesvm.png)

### 5. Install Kubeadm, Kubelet and Kubectl

#### 5.1 Update package repo

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

#### 5.2 Download the public signing key for the Kubernetes package repositories

```bash
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

```

#### 5.3 Add the appropriate Kubernetes apt repository

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

```

#### 5.4 Install kubelet, kubeadm and kubectl, and pin their versions

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```
Enable Kubelet now:

```bash
sudo systemctl enable --now kubelet

```
## Nodes Configuration (Master node only)

Use **kubeadm** to init the master node (control plane node):

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

```
Execute the following commands to configure **kubectl**:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
Install a Container Network Interface (CNI):
- Install the Tigera Calico operator and custom resource definitions.
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml

```
- Install Calico by creating the necessary custom resource.

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/custom-resources.yaml
```

Remove taint in the master node so that pods can be scheduled on it:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

```

## Nodes Configuration (Worker nodes only)

Use kubeadm **join** command to let your worker nodes join the cluster:

```bash
kubeadm join <control_plane_endpoint>:<port> --token <token_value> --discovery-token-ca-cert-hash <discovery_token_ca_cert_hash>

```








