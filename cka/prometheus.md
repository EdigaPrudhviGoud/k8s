Sample Project
git clone https://github.com/isItObservable/Episode1---Kubernetes-Prometheus
cd Episode1---Kubernetes-Prometheus

Prometheus Installation
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext

Grafana Installation:
helm repo add grafana https://grafana.github.io/helm-charts 
helm repo update
helm install grafana grafana/grafana
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext

Note:Get your 'admin user & password' by running:
   kubectl get secret --namespace default prometheus-grafana -o jsonpath="{.data.admin-user}" | base64 --decode
   kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

Loki:
Reference:
https://grafana.com/docs/loki/latest/setup/install/helm/install-monolithic/

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

Note: For multiple Replicas:
nano values.yaml
loki:
  commonConfig:
    replication_factor: 3
  schemaConfig:
    configs:
      - from: "2024-04-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h
  pattern_ingester:
      enabled: true
  limits_config:
    allow_structured_metadata: true
    volume_enabled: true
  ruler:
    enable_api: true

minio:
  enabled: true
      
deploymentMode: SingleBinary

singleBinary:
  replicas: 3

# Zero out replica counts of other deployment modes
backend:
  replicas: 0
read:
  replicas: 0
write:
  replicas: 0

ingester:
  replicas: 0
querier:
  replicas: 0
queryFrontend:
  replicas: 0
queryScheduler:
  replicas: 0
distributor:
  replicas: 0
compactor:
  replicas: 0
indexGateway:
  replicas: 0
bloomCompactor:
  replicas: 0
bloomGateway:
  replicas: 0


helm install --values values.yaml loki grafana/loki

Install or upgrade the Loki deployment:
    helm install --values values.yaml loki grafana/loki
    helm upgrade --values values.yaml loki grafana/loki

Verify that Loki is running:
   kubectl get pods -n loki
