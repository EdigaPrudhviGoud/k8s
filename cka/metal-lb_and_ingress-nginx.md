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
Check ecternal-IP for Ingress-nginx-controller:
  k get svc -A

  NAMESPACE        NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                  AGE
  ingress-nginx    ingress-nginx-controller             LoadBalancer   10.97.19.86      10.11.12.240   80:31906/TCP             3h26m
________________________________________________________________________________________________________________________________________________________
Testing:

root@llm-master1:/home/master1/msmart_k8s# cat test_nginx.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: patashala.machint.com  # Replace with your domain
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80


2.
root@llm-master1:/home/master1/msmart_k8s# cat msmart_be.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: msmart-compute-be
  labels:
    app: msmart-compute-be
spec:
  replicas: 2  # Number of pod replicas
  selector:
    matchLabels:
      app: msmart-compute-be
  template:
    metadata:
      labels:
        app: msmart-compute-be
    spec:
      containers:
      - name: msmart-compute-be
        image: httpd:2.4
#        image: msmart_compute_be_dev:v1
#        imagePullPolicy: Never  # Use the local image
        ports:
        - containerPort: 5004  # Update to match your app's port
---
apiVersion: v1
kind: Service
metadata:
  name: msmart-compute-be-svc
  labels:
    app: msmart-compute-be
spec:
  selector:
    app: msmart-compute-be
  ports:
  - protocol: TCP
    port: 80       # Service port (external)
    targetPort: 80
    #targetPort: 5004  # Pod's container port
  type: ClusterIP  # Change to NodePort if you want external access without Ingress
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: msmart-compute-be-ingress
  annotations:
#    kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  rules:
  - host: dev-be.mvidya.ai  # Replace with your domain or IP
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: msmart-compute-be-svc
            port:
              number: 80


root@llm-master1:/home/master1/msmart_k8s# k get ingress
NAME                        CLASS   HOSTS                   ADDRESS        PORTS   AGE
msmart-compute-be-ingress   nginx   dev-be.mvidya.ai        10.11.12.240   80      19m
nginx-ingress               nginx   patashala.machint.com   10.11.12.240   80      9m48s


On Nginx server: server block of respective sub-domain mapping to ingress-nginx-controller external-ip(as shown above)

__________________________________________________________________________________________________________________________________

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
