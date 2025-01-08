k drain llm2 --ignore-daemonsets --delete-emptydir-data

Error:
root@llm-master1:/home/master1# k drain llm2 --ignore-daemonsets --delete-emptydir-data
node/llm2 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/calico-node-j7xs7, kube-system/kube-proxy-95njv
evicting pod kube-system/calico-kube-controllers-5745477d4d-82psj
error when evicting pods/"calico-kube-controllers-5745477d4d-82psj" -n "kube-system" (will retry after 5s): Cannot evict pod as it would violate the pod's disruption budget.
evicting pod kube-system/calico-kube-controllers-5745477d4d-82psj

Solution:
k drain llm2 --force --ignore-daemonsets --delete-emptydir-data


k delete node llm2

ssh <username>@llm2
sudo systemctl stop kubelet
