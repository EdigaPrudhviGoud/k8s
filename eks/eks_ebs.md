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
eksctl addon --name aws-ebs-csi-driver --cluster <cluster-name> --service-account-role-arn arn:aws:iam:401231317770:role/AmazonEKS_EBS_CSI_Driver --force
#401231317770: The AWS account ID that owns the IAM role.
To find the 12 digit AWS account ID using the AWS CLI, you can run this command:
aws sts get-caller-identity
```
vi ebs-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
  #You can also make it the default StorageClass: Uncomment the below two lines
  #annotations:
  #  storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3   # You can also use gp2, io1, sc1, st1

kubectl apply -f ebs-storageclass.yaml
```

