Kubernetes Cluster Setup Using Kubeadm (Debian/Ubuntu)
This guide walks you through setting up a Kubernetes cluster on Debian/Ubuntu (like AWS EC2 instances) using kubeadm. It covers both master and worker node configuration.


Tum ec2 me ye port dena : 🔹 Master Node Ports:

22 – SSH (your IP)

6443 – API server (from nodes)

10250, 10257, 10259 – K8s components (from nodes)

2379, 2380 – etcd (from nodes)

80, 443 – (optional HTTP/HTTPS)

🔹 Worker Node Ports:

22 – SSH (your IP)

10250 – Kubelet (from master)

30000–32767 – NodePort (optional)

80, 443 – (optional)

179 – Calico (if used)

Prerequisites
Debian/Ubuntu OS (18.04 or later)

sudo privileges on all nodes

Internet access on all nodes

Recommended instance type: t2.medium or better (AWS EC2)

All instances must be in the same security group



AWS Security Group Setup
In AWS Console, go to EC2 > Security Groups.

Create a security group, e.g., Kubernetes-Cluster-SG.

Add inbound rules:

SSH: TCP port 22, Source: your IP (or 0.0.0.0/0 for open)

Kubernetes API: TCP port 6443, Source: all worker node IPs or 0.0.0.0/0 for testing

Attach this security group to all master and worker nodes.


Fresh Kubernetes Setup
Common commands (Run on Master and Worker nodes both)


sudo swapoff -a
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/k8s.conf
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release apt-transport-https

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

sudo apt-get update
sudo apt-get install -y containerd.io

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet







step 2  :-  Master Node Specific Commands (Sirf master node pe chalayein)

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml

echo "Cluster initialized. Get join command below:"
kubeadm token create --print-join-command



Note: Jo output kubeadm token create --print-join-command se milega, usko copy kar ke worker nodes pe use karna hai.

steps 3 : -  Worker Node Specific Commands (Sirf worker nodes pe chalayein)

example :- sudo kubeadm join 172.31.80.195:6443 --token 4qny1l.nyievcrd6l806pr8 --discovery-token-ca-cert-hash sha256:fae32570952e3468309f0a31ca316d512eb16339f088aef6dd806a9ac4b64a35



Verify on Master Node:  kubectl get nodes


example Expected output:
NAME           STATUS   ROLES           AGE    VERSION
master-node    Ready    control-plane   XXm    v1.29.15
worker-node1   Ready    <none>          XXm    v1.29.15
worker-node2   Ready    <none>          XXm    v1.29.15






