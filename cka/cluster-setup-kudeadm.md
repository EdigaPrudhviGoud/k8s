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
  --control-plane-endpoint "10.11.12.40:6443" \
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

******)Join the Worker Nodes to the Cluster

In the Control Plane Node, create the token and copy the kubeadm join command (NOTE: The join command can also be found in the output from kubeadm init command):

kubeadm token create --print-join-command
In both Worker Nodes, paste the kubeadm join command to join the cluster. Use sudo to run it as root:

sudo kubeadm join ...
In the Control Plane Node, view cluster status (Note: You may have to wait a few moments to allow all nodes to become ready):

kubectl get nodes

-> ADD Label: 
Node-Name: appian-dev5.machint.com
k label node appian-dev5.machint.com node-role.kubernetes.io/worker=


*******)Join the master Nodes to the Cluster: 10.11.12.40: First master node
kubeadm token create --print-join-command
	kubeadm join 10.11.12.40:6443 --token isd5be.1y6zyjll2tza9elp --discovery-token-ca-cert-hash sha256:e52fd110ce6752c5d66941181de793329247898ee9d0dfc5cbbc3c234b9ee42d 
For the above command add ---->>> --control-plane --certificate-key <certificate-key>

Certificate key: <certificate-key> (you will need to retrieve this from the first master node)
root@llm-master1:/home/master1/Documents# kubeadm init phase upload-certs --upload-certs
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
4ecc927103303781d4eed39fafdb0b2d39bef2d56e5fb9818150c9e31ed23084

______________________________________________________________________________________________________________________________________________________________________________________________________
************************************************************************************************************************************************************************************
ERRORS:
1.
Master1>
kubectl get no
NAME          STATUS   ROLES           AGE     VERSION
llm-master1   Ready    control-plane   5h8m    v1.32.0
master2       Ready    <none>          3m17s   v1.32.0

Master2> kubeadm join 10.11.12.40:6443 --token isd5be.1y6zyjll2tza9elp --discovery-token-ca-cert-hash sha256:e52fd110ce6752c5d66941181de793329247898ee9d0dfc5cbbc3c234b9ee42d --control-plane --certificate-key <certificate-key>

ror":"rpc error: code = FailedPrecondition desc = etcdserver: can only promote a learner member which is in sync with leader"}
I1227 23:50:51.421128   14300 etcd.go:584] [etcd] Promoting the learner c7f8df6beecbaa17 failed: etcdserver: can only promote a learner member which is in sync with leader
{"level":"warn","ts":"2024-12-27T23:50:51.920285+0530","logger":"etcd-client","caller":"v3@v3.5.16/retry_interceptor.go:63","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0006fa3c0/10.11.12.40:2379","attempt":0,"error":"rpc error: code = FailedPrecondition desc = etcdserver: can only promote a learner member which is in sync with leader"}
I1227 23:50:51.920440   14300 etcd.go:584] [etcd] Promoting the learner c7f8df6beecbaa17 failed: etcdserver: can only promote a learner member which is in sync with leader
{"level":"warn","ts":"2024-12-27T23:50:52.421079+0530","logger":"etcd-client","caller":"v3@v3.5.16/retry_interceptor.go:63","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0006fa3c0/10.11.12.40:2379","attempt":0,"error":"rpc error: code = FailedPrecondition desc = etcdserver: can only promote a learner member which is in sync with leader"}
I1227 23:50:52.421206   14300 etcd.go:584] [etcd] Promoting the learner c7f8df6beecbaa17 failed: etcdserver: can only promote a learner member which is in sync with leader
{"level":"warn","ts":"2024-12-27T23:50:52.920619+0530","logger":"etcd-client","caller":"v3@v3.5.16/retry_interceptor.go:63","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0006fa3c0/10.11.12.40:2379","attempt":0,"error":"rpc error: code = FailedPrecondition desc = etcdserver: can only promote a learner member which is in sync with leader"}
I1227 23:50:52.920758   14300 etcd.go:584] [etcd] Promoting the learner c7f8df6beecbaa17 failed: etcdserver: can only promote a learner member which is in sync with leader



Solution:
ON ALL NODES: 
ufw disable
ufw reload 

***************************************************************************************************************************************8
2.Just CHecking: etcd is in sync with master2
Master1>
sudo apt update
sudo apt install etcd-client 

Check Leader:
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint status --write-out=table

*************************************************************************************************************************************
Warning: Check etcd logs of both
{"level":"warn","ts":"2024-12-28T08:48:44.522403Z","caller":"rafthttp/probing_status.go:82","msg":"prober found high clock drift","round-tripper-name":"ROUND_TRIPPER_RAFT_MESSAGE","remote-peer-id":"4df3de4b88bd271","clock-drift":"1.123978766s","rtt":"15.900899ms"}
{"level":"warn","ts":"2024-12-28T08:48:44.547948Z","caller":"rafthttp/probing_status.go:82","msg":"prober found high clock drift","round-tripper-name":"ROUND_TRIPPER_SNAPSHOT","remote-peer-id":"4df3de4b88bd271","clock-drift":"1.131406653s","rtt":"1.043106ms"}

Solution: sudo date --set="2024-12-28 02:12:50"

