1. What is the VPC CNI plugin in EKS, and why is it used?
Answer: The VPC CNI plugin is a Kubernetes network plugin designed specifically for EKS. It allows each pod to get a direct IP address from the VPC. With VPC CNI, pods can communicate with each other, other services in the VPC, and other AWS resources (like EC2 instances, RDS, S3, etc.) without going through a NAT gateway or other proxies. The plugin uses ENIs (Elastic Network Interfaces) to associate IP addresses with pods.
It’s used to:
Allow Kubernetes pods to use VPC-native networking.
Provide pod-level networking with direct access to AWS services and other resources.
Enable advanced networking features like security groups and private IPs for pods.

Services:
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP/NodePort/LoadBalancer/ExternalName(The ExternalName service type maps a service to an external DNS name)
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30007 #This line for NodePort
______________________________________________________________________________________________________________________________________________________
Ingress in AWS EKS

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sample-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing  # ALB scheme (internet-facing or internal)
    # Optional: Enable SSL termination (assuming an SSL certificate is configured)
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account-id:certificate/certificate-id
    alb.ingress.kubernetes.io/target-type: ip   # This ensures traffic is routed to pod IPs

spec:
  ingressClassName: alb  # This field tells Kubernetes to use the ALB ingress controller for this Ingress
  rules:
  - host: your-domain.com  # Replace with your actual domain
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: sample-app
            port:
              number: 80
-------------------------------------------------------------------------------------------------------------------------------------
Network Policy allows you how a pod is allowed to communicate with: other pods,Namespaces, Ip blocks. It is disabled by default. Network policies are implemented by CNi.Network policy by default is deny
Ingress-traffic to the pod
Egress-traffic from the pod

Ingress Example:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-ingress
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: frontend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
      ports:
        - protocol: TCP
          port: 8080
_________________________________________________________
Egress Example:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-egress
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: frontend
  egress:
    - to:
        - podSelector/ipBlock(For this we have cdir: 10.0.0.0/24):
            matchLabels:
              app: my-service
      ports:
        - protocol: TCP
          port: 80
____________________________________________________________________________________________________
IPv6
Pods and services in eks have IPv6
Nodes have both IPv4 and IPv6
IPv6 only available at cluster creation. Pods in pvt subnets use an Egress-only IGW to communicate with external IPv6. Pods in pvt subnets uses primary network interface of node(then to NAT Gateway) to communicate with to communicate with external IPv4.

