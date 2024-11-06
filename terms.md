1.Cluster: A cluster is the actual Kubernetes environment where your workloads run.
Context: A context is a configuration that tells kubectl which cluster to communicate with, along with the necessary authentication details.

2.The core integral parts of a container:
    i)Namespaces: Namespaces are a feature of the Linux kernel that provides isolation for processes. They allow multiple containers to run on the same host without interfering with each other by providing separate views of system resources (like process IDs, network interfaces, etc.).
    ii)Control Groups (cgroups): Control Groups are another feature of the Linux kernel that manage and limit the resource usage of processes. They help ensure that a container does not consume more than its allocated share of system resources, such as CPU, memory, and I/O.
    iii)Capabilities
    iV) Linux security modules


3.ReplicaSet: A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. It is used by Deployment to perform that purpose in a higher-level concept.

4. A Helm chart is a package that contains all the necessary resources to deploy an application to a k8s cluster. Helm takes the complexity out of the process and makes Kubernetes deployment extremely simple and easy to handle.

5.Kubernetes controller manager is a control loop that monitors and regulates the state of a Kubernetes cluster

6.Scheduler is responsible for scheduling pods on specific nodes as per the user-defined conditions

6.Container Insights summarizes metrics and logs for Containerized applications. EKS Control Plane logs are not collected by CloudWatch container Insights.
Which AWS service can you use to troubleshoot EKS Control Plane logs?  Cloudwatch logs

kube-apiserver : The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

GuardDuty is a threat detection service that continuously monitors for malicious activity and unauthorized behavior for Amazon EKS Runtime and Log Monitoring Protection, S3 data events protection, Malware scannin protection, RDS login monitoring, and Lambda Networki Activity monitoring within your AWS accounts and workloads.
AWS Security Hub

______________________________________________________________________________________________________________________________________________
Resource	        Scope	        Purpose
Role	            Namespace	    Defines permissions within a specific namespace.
RoleBinding	        Namespace	    Grants the permissions of a Role to a user or service account within a specific namespace.
ClusterRole	        Cluster-wide	Defines permissions that can be applied cluster-wide, across all namespaces.
ClusterRoleBinding	Cluster-wide	Grants the permissions of a ClusterRole to a user or service account across the entire cluster.
________________________________________________________________________________________________________________________________________

***)Taints are applied to nodes to prevent pods from being scheduled on them unless the pod has a matching toleration.
Tolerations are applied to pods to allow them to be scheduled on nodes that have matching taints.
This mechanism is useful for managing resource allocation, workload isolation, and node-specific constraints (e.g., special hardware like GPUs).


PodAffinity Example:
    Stateful applications (like databases) might need to be placed on the same node or in the same zone to optimize for high throughput or low latency.
    Microservices that need to communicate frequently might benefit from being placed together to reduce network latency.

PodAntiAffinity Example:
    High availability: Ensuring that multiple replicas of a service are spread across multiple nodes or availability zones.
    Fault tolerance: Ensuring that critical services or workloads are not placed on the same node to reduce the risk of a single point of failure.

CoreDNS is the default DNS service used in Amazon EKS (Elastic Kubernetes Service) clusters. It provides DNS resolution for Kubernetes services, allowing your Kubernetes workloads to discover and communicate with other services and pods within the cluster. CoreDNS replaces the older kube-dns service and provides better performance, extensibility, and flexibility.
    Resource isolation: Keeping certain workloads (e.g., batch jobs, or low-priority jobs) away from others to avoid resource contention.
