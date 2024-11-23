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
------------------------------------------------------------------
CAS(Cluster Autoscalar)
