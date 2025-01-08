How to Deploy Kafka on Kubernetes:
Step 1: Create Project Directory
  mkdir kafka
  cd kafka
*****************************************************************************************************
Step 2: Create Network Policy
  nano kafka-network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: kafka-network
spec:
  ingress:
    - from:
        - podSelector:
            matchLabels:
              network/kafka-network: "true"
  podSelector:
    matchLabels:
      network/kafka-network: "true"

***********************************************************************************************************
Step 3: Create ZooKeeper Stateful Set
  nano zookeeper-stateful-set.yaml 

apiVersion: v1
kind: PersistentVolume
metadata:
  name: zookeeper-data-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data/zookeeper/data  # Matches the StatefulSet's `volumeMounts` path for `zookeeper-data`

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: zookeeper-log-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data/zookeeper/log  # Matches the StatefulSet's `volumeMounts` path for `zookeeper-log`

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    service: zookeeper
  name: zookeeper
spec:
  serviceName: zookeeper
  replicas: 1
  selector:
    matchLabels:
      service: zookeeper
  template:
    metadata:
      labels:
        network/kafka-network: "true"
        service: zookeeper
    spec:
      securityContext:
        fsGroup: 1000
      enableServiceLinks: false
      containers:
        - name: zookeeper
          imagePullPolicy: Always
          image: zookeeper
          ports:
            - containerPort: 2181
          env:
            - name: ZOOKEEPER_CLIENT_PORT
              value: "2181"
            - name: ZOOKEEPER_DATA_DIR
              value: "/data"    ## Matching the default zoo.cfg path
            - name: ZOOKEEPER_LOG_DIR
              value: "/datalog"    ## Matching the default zoo.cfg path
            - name: ZOOKEEPER_SERVER_ID
              value: "1"    ## For a single node setup
          resources: {}
          volumeMounts:
            - mountPath: /data
              name: zookeeper-data
            - mountPath: /datalog
              name: zookeeper-log
      hostname: zookeeper
      restartPolicy: Always
      affinity:  # Node Affinity to ensure pods are scheduled on the same node
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/hostname
                    operator: In
                    values:
                      - appian-dev5.machint.com  # Replace with your node's name
  volumeClaimTemplates:
    - metadata:
        name: zookeeper-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1024Mi
        storageClassName: standard  # This should match the StorageClass if applicable
    - metadata:
        name: zookeeper-log
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1024Mi
        storageClassName: standard  # This should match the StorageClass if applicable
---
apiVersion: v1
kind: Service
metadata:
  labels:
    service: zookeeper
  name: zookeeper
spec:
  ports:
    - name: "2181"
      port: 2181
      targetPort: 2181
  selector:
    service: zookeeper
*******************************************************************************************************************************************
Step 4: Create Kafka Stateful Set along with pv and svc
  nano kafka-stateful-set.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kafka-data-pv
spec:
  capacity:
    storage: 1Gi  # Adjust the size as per your needs
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain  # or Delete if you want the volume deleted on PVC deletion
  storageClassName: standard  # Should match the PVC's StorageClass if applicable
  local:
    path: /mnt/data/kafka/data  # Path to the local disk or directory (if using local storage)
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - appian-dev5.machint.com  # Replace with your node's name if using node affinity
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    service: kafka
  name: kafka
spec:
  serviceName: kafka
  replicas: 1
  selector:
    matchLabels:
      service: kafka
  template:
    metadata:
      labels:
        network/kafka-network: "true"
        service: kafka
    spec:
      securityContext:
        fsGroup: 1000
      enableServiceLinks: false
      containers:
      - name: kafka
        imagePullPolicy: IfNotPresent
        image: bitnami/kafka:latest
        ports:
          - containerPort: 29092
          - containerPort: 9092
        env:
          - name: KAFKA_ADVERTISED_LISTENERS
            value: "INTERNAL://:29092,LISTENER_EXTERNAL://:9092"
          - name: KAFKA_AUTO_CREATE_TOPICS_ENABLE
            value: "true"
          - name: KAFKA_INTER_BROKER_LISTENER_NAME
            value: "INTERNAL"
          - name: KAFKA_LISTENERS
            value: "INTERNAL://:29092,LISTENER_EXTERNAL://:9092"
          - name: KAFKA_LISTENER_SECURITY_PROTOCOL_MAP
            value: "INTERNAL:PLAINTEXT,LISTENER_EXTERNAL:PLAINTEXT"
          - name: KAFKA_ZOOKEEPER_CONNECT
            value: "zookeeper:2181"
        resources: {}
        volumeMounts:
          - mountPath: /var/lib/kafka/
            name: kafka-data
      hostname: kafka
      restartPolicy: Always
  volumeClaimTemplates:
    - metadata:
        name: kafka-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
        storageClassName: standard  # Explicitly match the PV's StorageClass
---
apiVersion: v1
kind: Service
metadata:
  labels:
    service: kafka
  name: kafka
spec:
  clusterIP: None
  selector:
    service: kafka
  ports:
    - name: internal
      port: 29092
      targetPort: 29092
    - name: external
      port: 30092
      targetPort: 9092
***********************************************************************************************************************************************
Step 5: Test Kafka Deployment
  nano kcat-deployment.yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: kcat
  labels:
    app: kcat
spec:
  selector:
    matchLabels:
      app: kcat
  template:
    metadata:
      labels:
        app: kcat
    spec:
      containers:
        - name: kcat
          image: edenhill/kcat:1.7.0
          command: ["/bin/sh"]
          args: ["-c", "trap : TERM INT; sleep 1000 & wait"]

kubectl apply -f kcat-deployment.yaml
kubectl exec --stdin --tty [pod-name] -- /bin/sh
8. Enter the command below to send Kafka a test message to ingest:
        echo "Test Message" | kcat -P -b kafka:29092 -t testtopic -p -1      #kafka=svc-name and port of svc
  If successful, the command prints no output.

9. Switch to the consumer role and query Kafka for messages by typing:
        kcat -C -b kafka:29092 -t testtopic -p -1
    The test message appears in the output.:  Test Message



******************************************************************************************************************************
