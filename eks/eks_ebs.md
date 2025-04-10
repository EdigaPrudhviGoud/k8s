https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html

Store Kubernetes volumes with Amazon EBS

The Amazon EBS CSI plugin requires IAM permissions to make calls to AWS APIs on your behalf. If you don’t do these steps, attempting to install the add-on and running kubectl describe pvc will show failed to provision volume with StorageClass along with a could not create volume in EC2: UnauthorizedOperation error. For more information, see Set up driver permission on GitHub.

Step 1: Create an IAM role
The following procedure shows you how to create an IAM role and attach the AWS managed policy to it. To implement this procedure, you can use one of these tools:
i)eksctl
ii)AWS Management Console
iii)AWS CLI
```
i)Install EKSCTL
ii)eksctl utils associate-oidc-provider --cluster <cluster-name> --aprove
eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster <my-cluster> \
        --role-name AmazonEKS_EBS_CSI_DriverRole \
        --role-only \
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
        --approve
eksctl addon --name aws-ebs-csi-driver --cluster <cluster-name> --service-account-role-arn arn:aws:iam:401231317770:role/AmazonEKS_EBS_CSI_Driver --fo
```
```
iii)
#1)View your cluster’s OIDC provider URL. Replace my-cluster with your cluster name. If the output from the command is None, review the Prerequisites.
aws eks describe-cluster --name my-cluster --query "cluster.identity.oidc.issuer" --output text
#O/P:   https://oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE

#2)Create the IAM role, granting the AssumeRoleWithWebIdentity action.
#a)Copy the following contents to a file that’s named aws-ebs-csi-driver-trust-policy.json. Replace 111122223333 with your account ID. Replace EXAMPLED539D4633E53DE1B71EXAMPLE and region-code with the values returned in the previous step.
{
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
          },
          "Action": "sts:AssumeRoleWithWebIdentity",
          "Condition": {
            "StringEquals": {
              "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com",
              "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
            }
          }
        }
      ]
    }

#b)Create the role. You can change AmazonEKS_EBS_CSI_DriverRole to a different name. If you change it, make sure to change it in later steps.
aws iam create-role \
      --role-name AmazonEKS_EBS_CSI_DriverRole \
      --assume-role-policy-document file://"aws-ebs-csi-driver-trust-policy.json"

#c)Attach a policy. AWS maintains an AWS managed policy or you can create your own custom policy. Attach the AWS managed policy to the role with the following command.
aws iam attach-role-policy \
      --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
      --role-name AmazonEKS_EBS_CSI_DriverRole
```
