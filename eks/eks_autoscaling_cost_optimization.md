HPA---(Check CPU or Memory)-->metrics server---->Kubelet's(worker node1,node2,node3)
HPA ---> Prometheus Adapter---> Prometheus(checks any metric)--->
KEDA(Kubernetes Event Driven Autoscaling)-->dynamoDb,kinesis,SQS,CW,MongoDB,RabbitMQ,Apache Kafka....

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

2.HPA WITH Prometheus
1. Prometheus Adapter Installation
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus-adapter prometheus-community/prometheus-adapter --namespace monitoring
2.HPA YAML File
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: External
      external:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "50"

Explanation:
    metric.name:
        http_requests_per_second is a custom metric collected by Prometheus, which tracks the number of HTTP requests per second.
        Replace this with your desired custom metric's name.
    target.averageValue:
        Specifies the target value for the metric (e.g., scale up if HTTP requests exceed 50 requests/second).
    scaleTargetRef:
        This points to the Kubernetes resource (e.g., Deployment) you want to scale.

HPA DEMO:
kubectl get po -n kube-system #check for metrics-server pod 
eks-node-viewer:Application installed for visualize the utilization of nodes
kubectl top node #to check CPU(cores),cpu%,Memory(bytes),memory%
Steps:
kubectl run my-deployment --image=nginx --restart=Always --replicas=1
kubectl set resources deployment my-deployment \
--limits=cpu=500m,memory=256Mi \
--requests=cpu=250m,memory=128Mi
kubectl autoscale deployment my-deployment --cpu-percent=50 --min=1 --max=5 #For existing deployment
kubectl get hpa -A
------------------------------------------------------------------
CAS(Cluster Autoscalar): The Cluster Autoscaler in Amazon EKS (Elastic Kubernetes Service) is a Kubernetes component that automatically adjusts the size of your cluster by adding or removing EC2 instances (nodes) based on the pending pods' scheduling requirements.
The Amazon EC2 Auto Scaling group (ASG) for your cluster must be properly configured.
Configuration using helm or yaml:
spec:
  containers:
  - image: k8s.gcr.io/autoscaler/cluster-autoscaler:v1.27.3
    name: cluster-autoscaler
    command:
    - ./cluster-autoscaler
    - --v=4
    - --stderrthreshold=info
    - --cloud-provider=aws
    - --skip-nodes-with-local-storage=false
    - --nodes=1:10:<your-autoscaling-group-name>
    - --cluster-name=<your-cluster-name>

k get deployment -n kube-system cluster-autoscalar-aws-cluster-autoscalar
___________________________________________________________________________
Karpenter:
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  requirements:
    - key: "instance-type"
      operator: In
      values:
        - c5.large
        - m5.large
        - m5.xlarge
  limits:
    resources:
      cpu: 1000
      memory: 2000Gi
  provider:
    instanceProfile: KarpenterNodeInstanceProfile
  ttlSecondsAfterEmpty: 30

kubectl apply view-last-applied provisioner default -o yaml | yq

-----------------------------_____________________________________________________________
Cost Optimisation(Savings plans, spot instances, Cost report with KubeCost):

____________________________________________________________________________________

