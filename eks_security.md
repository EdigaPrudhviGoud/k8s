1.Image Security: ECR, ECR public
Whenever we push image to ECR then amazon inspector scan image OS and programming language packages.

Image -> ECR -> Amazon inspector scans images -> send report to AWS security hub

Dockerfile:
Add user to Dockerfile and make it a non-privileged user
Don't use --privileged flag when running a container
____________________________________________________________________________________________________________________________________________________________________________________________________________

2.User Access
The command you're trying to run is intended to update the kubeconfig file for interacting with an Amazon EKS (Elastic Kubernetes Service) cluster using AWS CLI. The command you're using is specifically designed for a situation where you've configured AWS Single Sign-On (SSO) and you want to authenticate and access an EKS cluster.

Here’s a breakdown of the command you provided: aws --profile aws-sso-tech eks update-kubeconfig --region eu-west-1 --name ssodemo

Explanation:

--profile aws-sso-tech: This specifies the AWS CLI profile to use for authentication. In your case, aws-sso-tech is the profile name you configured with AWS SSO credentials. You need to ensure this profile is set up correctly in your ~/.aws/credentials and ~/.aws/config files, and the SSO credentials are valid.

eks update-kubeconfig: This command updates the kubeconfig file used by kubectl to interact with the specified EKS cluster. It fetches the authentication credentials for the cluster and updates the local kubeconfig file so kubectl can work with it.

--region eu-west-1: This specifies the AWS region where your EKS cluster (ssodemo) is located. In this case, it’s in the eu-west-1 region (Ireland).

--name ssodemo: This specifies the name of the EKS cluster you want to update the kubeconfig for. In this case, it’s ssodemo.

Prerequisites: 1.AWS CLI Version: Make sure you’re using a recent version of the AWS CLI (v2), as AWS SSO and some of the newer features (like eks update-kubeconfig) are supported in AWS CLI v2. aws --version

2.AWS SSO Configuration: Ensure your AWS CLI profile aws-sso-tech is properly configured with AWS SSO. If you haven’t set it up, you need to run the following command to configure it: aws configure sso

3.Kubeconfig: The update-kubeconfig command will update or create a kubeconfig file (~/.kube/config by default) that kubectl uses to access the EKS cluster. Ensure kubectl is installed on your system. aws --profile aws-sso-tech eks update-kubeconfig --region eu-west-1 --name ssodemo

4.IAM Permissions: Ensure your IAM user/role has the necessary permissions to access the EKS cluster, such as: eks:DescribeCluster eks:ListClusters eks:GetClusterConfig

5.EKS Cluster: Verify that the EKS cluster (ssodemo) exists in the eu-west-1 region.

Log in to AWS SSO (if not already authenticated): steps: aws sso login --profile aws-sso-tech aws --profile aws-sso-tech eks update-kubeconfig --region eu-west-1 --name ssodemo kubectl config get-contexts

Similar to RBAC in K8S: Here will have IAM roles with attached policies, and these roles are attached to IAM users. Then we will a create role and attach it rolebinding with IAM USER.
______________________________________________________________________________________________________________________________________________________________________________________________________________
3. Security boundaries

______________________________________________________________________________________________________________________________________________________________________________________________________________
4.Network security: Network policies
In Amazon EKS (Elastic Kubernetes Service), Network Policies are used to control the traffic flow between pods, namespaces, or services within the Kubernetes cluster. They provide a way to define how pods can communicate with each other, based on rules that specify allowed or denied connections.

Network policies allow you to:
    Control ingress (incoming) and egress (outgoing) traffic for pods.
    Define traffic flow based on labels, IP blocks, and ports.
    Specify policies that allow or deny traffic.

To use network policies in an EKS cluster, you need to have a CNI (Container Network Interface) plugin that supports them. Amazon VPC CNI alone doesn't support network policies, so you'll need to install an additional CNI plugin like Calico or Cilium.


In Amazon EKS (Elastic Kubernetes Service), Network Policies are used to control the traffic flow between pods, namespaces, or services within the Kubernetes cluster. They provide a way to define how pods can communicate with each other, based on rules that specify allowed or denied connections.

Network Policies are implemented using Calico, Cilium, or other network plugins that support Kubernetes NetworkPolicy resources.

Below, I'll walk you through the basics of creating and applying network policies in an EKS cluster, along with some practical examples.
What Are Kubernetes Network Policies?

Network policies allow you to:

    Control ingress (incoming) and egress (outgoing) traffic for pods.
    Define traffic flow based on labels, IP blocks, and ports.
    Specify policies that allow or deny traffic.

To use network policies in an EKS cluster, you need to have a CNI (Container Network Interface) plugin that supports them. Amazon VPC CNI alone doesn't support network policies, so you'll need to install an additional CNI plugin like Calico or Cilium.
Steps to Enable Network Policies in EKS

    Install Calico on EKS:

    If you're using Amazon's default VPC CNI, it won't support network policies by default. You need to install Calico (or another CNI plugin that supports network policies). Follow the instructions in the Amazon EKS documentation to install Calico.

    Enable Network Policies in EKS:

    After installing Calico, Kubernetes network policies can be applied directly using the NetworkPolicy resource.

Basic Network Policy Structure
A NetworkPolicy is defined using YAML and contains the following key sections:
    podSelector: Specifies which pods the policy applies to.
    ingress: Rules defining which inbound traffic is allowed.
    egress: Rules defining which outbound traffic is allowed.
    policyTypes: Specifies whether the policy applies to ingress, egress, or both.

EX1:Let's say you have a frontend pod and you want it to accept traffic only from the backend pod.
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: frontend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
  policyTypes:
    - Ingress

Example 2: Denying All Ingress Traffic (Default Deny)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  ingress: []  # Empty ingress list means no ingress traffic is allowed
  policyTypes:
    - Ingress

Example 3: Allowing Egress to External IP Block
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-external-api
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: frontend
  egress:
    - to:
        - ipBlock:
            cidr: 192.168.1.0/24
  policyTypes:
    - Egress


Example 4: Allowing Ingress and Egress Between Specific Pods in Different Namespaces
If you have two namespaces, namespace-a and namespace-b, and you want to allow traffic between specific pods in both namespaces, you can specify cross-namespace communication.
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cross-namespace-traffic
  namespace: namespace-a
spec:
  podSelector:
    matchLabels:
      app: service-a
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: namespace-b
          podSelector:
            matchLabels:
              app: service-b
  policyTypes:
    - Ingress

Summary of Key Fields
podSelector	Selects which pods the policy applies to.
ingress	Defines allowed inbound traffic to the pod.
egress	Defines allowed outbound traffic from the pod.
from / to	Specifies sources or destinations for traffic.
ipBlock	Allows traffic from a specific IP range.
namespaceSelector	Restricts traffic based on the namespace of the source.
policyTypes	Specifies whether the policy applies to ingress, egress, or both.

Conclusion
Network policies in Amazon EKS, when used with a compatible CNI plugin like Calico, allow you to define precise rules for controlling traffic between pods, namespaces, and external resources. The ability to define both ingress and egress policies gives you fine-grained control over how your services communicate, improving security and fault tolerance.
