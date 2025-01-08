1.Instll INgress-Nginx
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

2.Instal MetalLb:
https://metallb.io/installation/

Installation by manifest:
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml

3.Configure MetalLB: MetalLB needs to be configured with a pool of IP addresses from which it can assign an external IP. You can create a ConfigMap for MetalLB with an IP range:
nano metal-lb_config.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
  namespace: metallb-system
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.11.12.240-10.11.12.250  
# Specify an available IP range in your host network
#The above IP address will be used by the services of Type-Load Balancers


Uninstall and Re-install nginx-ingress:
helm uninstall ingress-nginx -n ingress-nginx
kubectl get all -n ingress-nginx

helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

kubectl patch svc ingress-nginx-controller -n ingress-nginx \
  -p '{"spec": {"loadBalancerIP": "10.11.12.240"}}'

kubectl get svc -n ingress-nginx

