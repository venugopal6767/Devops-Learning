Step 1: Run on All Nodes (Master + Workers)
-------------------------------------------
1. Update and upgrade all nodes
2. Disable SWAP and load Kernal Modules
3. Install containered
4. Install Kubeadm kubelet and kubectl
5. Initialize the control Plane and save the join command
6. configure Kubectl on Master
7. Install Calico CNI Plugin
8. join Worker nodes by running the join command



# 1. Update 
sudo apt update && sudo apt upgrade -y
# 2. Disable Swap (required by kubelet)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab 
# 3. Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
# 4. Set sysctl parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system


Step 2: Install containerd
# 1. Install containerd dependencies
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

# 2. Add Docker GPG key and repo
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 3. Install containerd
sudo apt update && sudo apt install -y containerd.io

# 4. Generate default config
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

# 5. Enable systemd cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# 6. Restart containerd
sudo systemctl restart containerd
sudo systemctl enable containerd


☸️ Step 3: Install Kubernetes
# 1. Add Kubernetes GPG key and repo
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

# 2. Install kubeadm, kubelet, kubectl
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl







sudo kubeadm join 172.31.91.6:6443 --token jvbag7.0fwtnn1rbpuw0bzn \
        --discovery-token-ca-cert-hash sha256:fe9a14d498c7b1e6837342837db08d92268948a03fac319967ed65f06958b0c8
