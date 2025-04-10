FOR BOTH MASTER NODE AND WORKER NODES

1. Disable Swap
```sudo swapoff -a ```
```sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab ```
2. Install Docker (using docker documentation is better)
```
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker

sudo usermod -aG docker $USER
newgrp docker  
```
3. Install K8’s Dependencies
```
sudo apt install -y apt-transport-https ca-certificates curl gnupg
```
4. Add K8 Repo
```
sudo mkdir -p /etc/apt/keyrings  # Fix "No such file" error
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
5. Install the trio (pick any version suitable )
```
sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl
```

NEXT STEPS ONLY FOR THE MASTER NODE 

1. Initialize Control Plane (follow the steps given after this command and save the token to join the slaves)
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16  
```
2. Install Flannel (CNI Plugin)
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
 3. Install NGINX Ingress 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
```



