1.Instll INgress-Nginx
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

2.Instal MetalLb:
Reference: https://metallb.io/installation/

step1>>>strictARP:true
    # see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system

Step2:Installation by manifest:
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml

Step3.Configure MetalLB: MetalLB needs to be configured with a pool of IP addresses from which it can assign an external IP. You can create a ConfigMap for MetalLB with an IP range:
nano ip_address_pool.yaml 
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.11.12.240-10.11.12.250
#Host IP Range
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system

_____________________________________________________________________________________________________________________________________________________________________________________________________________

Uninstall and Re-install nginx-ingress:
  helm uninstall ingress-nginx -n ingress-nginx
  kubectl get all -n ingress-nginx

  helm upgrade --install ingress-nginx ingress-nginx \
    --repo https://kubernetes.github.io/ingress-nginx \
    --namespace ingress-nginx --create-namespace

  kubectl patch svc ingress-nginx-controller -n ingress-nginx \
    -p '{"spec": {"loadBalancerIP": "10.11.12.240"}}'

  kubectl get svc -n ingress-nginx

  kubectl patch svc ingress-nginx-controller -n ingress-nginx \
    -p '{"spec": {"loadBalancerIP": null}}'

  kubectl patch svc ingress-nginx-controller -n ingress-nginx \
    -p '{"spec": {"loadBalancerIP": "10.11.12.200"}}'
