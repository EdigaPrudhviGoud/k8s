Connecting a Kubernetes pod running on Amazon EKS to an Amazon RDS (Relational Database Service) instance involves several steps, from setting up proper networking to ensuring the pod has the correct credentials to access the RDS database.
1. Set Up RDS Instance
      Database Configuration:
    Ensure the VPC of the RDS instance is the same or peered with the VPC where your EKS cluster resides.
    Set the RDS security group to allow traffic from the EKS nodes. The default RDS security group should allow inbound connections from the IP range or CIDR block of your EKS cluster.

2.Set Up IAM Permissions (Optional)
If your EKS pods are going to use IAM roles for authentication (instead of using a username/password), you need to configure the RDS IAM role authentication.
    Create an IAM Role:
        If you want to authenticate using IAM roles instead of traditional database credentials, you can set up IAM Database Authentication. Attach the appropriate IAM policy (rds-db:connect) to the role, and configure your RDS instance to accept IAM authentication.
    Associate IAM Role with EKS:
        You can assign the IAM role to a Kubernetes service account in EKS. Pods using this service account can authenticate to the RDS instance using IAM roles.

3. Configure the EKS Cluster Networking

Your EKS cluster must be able to communicate with the RDS instance. The RDS instance and the EKS nodes must be in the same VPC or connected via VPC peering.

    VPC: Ensure that both your EKS cluster and RDS instance are in the same VPC or, if not, that VPC peering is configured correctly.
    Subnets: Check if the subnets in which your RDS instance and EKS nodes reside have proper route tables and access to each other.

4. Configure Kubernetes Secrets (For Credentials)
You can store your database credentials securely in Kubernetes Secrets and then reference them in your pod configuration.
    kubectl create secret generic rds-credentials \
      --from-literal=username=your-db-username \
      --from-literal=password=your-db-password \
      --namespace your-namespace

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image
          env:
            - name: DB_HOST
              value: "your-db-endpoint.rds.amazonaws.com"
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: rds-credentials
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: rds-credentials
                  key: password
            - name: DB_PORT
              value: "3306"  # Use 5432 for PostgreSQL


import psycopg2
import os

connection = psycopg2.connect(
    host=os.getenv("DB_HOST"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD"),
    dbname="mydatabase",  # Replace with your actual database name
    port=os.getenv("DB_PORT")
)

cursor = connection.cursor()
cursor.execute("SELECT * FROM my_table")
result = cursor.fetchall()
print(result)
cursor.close()
connection.close()
