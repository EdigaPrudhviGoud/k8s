Commands : SetUp K8s HA Cluster (Updated)
Kubernetes Installation Process


************* Install Kubernetes on Master Node *************

1.Upgrade apt packages
sudo apt-get update

2.Create a configuration file for containerd:
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

3.Load modules:
sudo modprobe overlay
sudo modprobe br_netfilter


4.Set system configurations for Kubernetes networking:
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF


5.Apply new settings:
sudo sysctl --system

6.Install containerd:
sudo apt-get update && sudo apt-get install -y containerd


7.Create a default configuration file for containerd:
sudo mkdir -p /etc/containerd


8.Generate default containerd configuration and save to the newly created default file:
containerd config default | sudo tee /etc/containerd/config.toml


9.Restart containerd to ensure new configuration file usage:
sudo systemctl restart containerd
sudo systemctl enable containerd


10.Verify that containerd is running.
sudo systemctl status containerd


11.Disable swap:
sudo swapoff -a


12.Disable swap on startup in /etc/fstab:
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


13.Install dependency packages:
  sudo apt-get update && sudo apt-get install -y apt-transport-https curl ca-certificates gpg

14.Download and add the GPG key:
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


15.Add Kubernetes to the repository list:
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


16.Update package listings:
sudo apt-get update


17.Install Kubernetes packages (Note: If you get a dpkg lock message, just wait a minute or two before trying the command again):
sudo apt-get install -y kubelet kubeadm kubectl

18.Turn off automatic updates:
sudo apt-mark hold kubelet kubeadm kubectl


Log into both Worker Nodes to perform previous steps 1 to 18.

Initialize the Kubernetes cluster on the control plane node using kubeadm (Note: This is only performed on the Control Plane Node):
Initialize the First Control-Plane Node
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --upload-certs 

Set kubectl access:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubeadm join 10.11.12.40:6443 --token isd5be.1y6zyjll2tza9elp \
	--discovery-token-ca-cert-hash sha256:e52fd110ce6752c5d66941181de793329247898ee9d0dfc5cbbc3c234b9ee42d 


Test access to cluster:
kubectl get nodes
kubectl get po -A 
Cluster Status: Node Notready and Coredns->Pending state

Install the Calico Network Add-On -> On the Control Plane Node, install Calico Networking:
Reference: https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

NOTE: If you are using pod CIDR 192.168.0.0/16, skip to the next step. If you are using a different pod CIDR with kubeadm, no changes are required — Calico will automatically detect the CIDR based on the running configuration. For other platforms, make sure you uncomment the CALICO_IPV4POOL_CIDR variable in the manifest and set it to the same value as your chosen pod CIDR.

kubectl apply -f calico.yaml


Wait for 2-4 Min and Check the status of the control plane node and Coredns:
kubectl get nodes
kubectl get po -A

Join the Worker Nodes to the Cluster

In the Control Plane Node, create the token and copy the kubeadm join command (NOTE: The join command can also be found in the output from kubeadm init command):

kubeadm token create --print-join-command
In both Worker Nodes, paste the kubeadm join command to join the cluster. Use sudo to run it as root:

sudo kubeadm join ...
In the Control Plane Node, view cluster status (Note: You may have to wait a few moments to allow all nodes to become ready):

kubectl get nodes
