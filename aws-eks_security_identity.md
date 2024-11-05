The command you're trying to run is intended to update the kubeconfig file for interacting with an Amazon EKS (Elastic Kubernetes Service) cluster using AWS CLI. The command you're using is specifically designed for a situation where you've configured AWS Single Sign-On (SSO) and you want to authenticate and access an EKS cluster.

Here’s a breakdown of the command you provided:
aws --profile aws-sso-tech eks update-kubeconfig --region eu-west-1 --name ssodemo

Explanation:

    --profile aws-sso-tech: This specifies the AWS CLI profile to use for authentication. In your case, aws-sso-tech is the profile name you configured with AWS SSO credentials. You need to ensure this profile is set up correctly in your ~/.aws/credentials and ~/.aws/config files, and the SSO credentials are valid.

    eks update-kubeconfig: This command updates the kubeconfig file used by kubectl to interact with the specified EKS cluster. It fetches the authentication credentials for the cluster and updates the local kubeconfig file so kubectl can work with it.

    --region eu-west-1: This specifies the AWS region where your EKS cluster (ssodemo) is located. In this case, it’s in the eu-west-1 region (Ireland).

    --name ssodemo: This specifies the name of the EKS cluster you want to update the kubeconfig for. In this case, it’s ssodemo.

Prerequisites:
1.AWS CLI Version: Make sure you’re using a recent version of the AWS CLI (v2), as AWS SSO and some of the newer features (like eks update-kubeconfig) are supported in AWS CLI v2.
    aws --version

2.AWS SSO Configuration: Ensure your AWS CLI profile aws-sso-tech is properly configured with AWS SSO. If you haven’t set it up, you need to run the following command to configure it:
    aws configure sso

3.Kubeconfig: The update-kubeconfig command will update or create a kubeconfig file (~/.kube/config by default) that kubectl uses to access the EKS cluster. Ensure kubectl is installed on your system.
    aws --profile aws-sso-tech eks update-kubeconfig --region eu-west-1 --name ssodemo

4.IAM Permissions: Ensure your IAM user/role has the necessary permissions to access the EKS cluster, such as:
    eks:DescribeCluster
    eks:ListClusters
    eks:GetClusterConfig

5.EKS Cluster: Verify that the EKS cluster (ssodemo) exists in the eu-west-1 region.

______________________________________________________________________________________________________________________________________________
Log in to AWS SSO (if not already authenticated):
steps:
aws sso login --profile aws-sso-tech
aws --profile aws-sso-tech eks update-kubeconfig --region eu-west-1 --name ssodemo
kubectl config get-contexts


Similar to RBAC in K8S:
Here will have IAM roles with attached policies, and these roles are attached to IAM users. Then we will a create role and attach it rolebinding with IAM USER.
