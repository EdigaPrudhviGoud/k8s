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
        image: msmart_compute_be_dev:v1
        imagePullPolicy: Never  # Use the local image
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
    targetPort: 5004  # Pod's container port
  type: ClusterIP  # Change to NodePort if you want external access without Ingress
---
#apiVersion: networking.k8s.io/v1
#kind: Ingress
#metadata:
#  name: msmart-compute-be-ingress
#  annotations:
#    nginx.ingress.kubernetes.io/rewrite-target: /
#spec:
#  rules:
#  - host: msmart.example.com  # Replace with your domain or IP
#    http:
#      paths:
#      - path: /
#        pathType: Prefix
#        backend:
#          service:
#            name: msmart-compute-be-svc
#            port:
#              number: 80
