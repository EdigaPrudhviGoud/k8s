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
------------------------------------------------------------------
CAS(Cluster Autoscalar)
