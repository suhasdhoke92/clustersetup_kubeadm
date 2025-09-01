# clustersetup_kubeadm
minimum command to set up 2 nodes cluster with 

# Kubernetes Cluster Setup with kubeadm (v1.34) on Ubuntu

Set up a Kubernetes cluster with two nodes using kubeadm (v1.34) on Ubuntu.

I followed the official Kubernetes documentation pages below. These docs are detailed, but sometimes we just need the minimal commands to get started. Here are the essential commands that worked for me. 🙂

- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
- https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

---

## On the Control Plane Node

```bash
# Enable IP forwarding
sudo bash -c 'echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/k8s.conf'
sudo sysctl --system

# Install dependencies and Kubernetes packages 
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

# Initialize the control plane
sudo kubeadm init --apiserver-advertise-address=<control-plane-ip> --pod-network-cidr=<pod-cidr>

# Example
sudo kubeadm init --apiserver-advertise-address=172.16.50.123 --pod-network-cidr=10.244.0.0/16

# Set up kubectl (this command will get in above command output)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install Flannel CNI plugin (followed one of CNI plugin from k8s page which is quite easy to deploy)
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

## On the Worker Node

```bash
# Install dependencies and Kubernetes packages
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

# Join the cluster (this command taken from output of kubeadm init command which we executed on Controller )
sudo kubeadm join <control-plane-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>

# Example: 
sudo kubeadm join 172.16.50.123:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

---

## Verify the Cluster

```bash
kubectl get nodes
kubectl
