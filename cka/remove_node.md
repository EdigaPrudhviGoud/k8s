k drain llm2 --ignore-daemonsets --delete-emptydir-data

k delete node llm2

ssh <username>@llm2
sudo systemctl stop kubelet
