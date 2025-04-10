https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html
https://www.eksworkshop.com/docs/fundamentals/storage/ebs/ebs-csi-driver
```
i)Install EKSCTL
ii)eksctl utils associate-oidc-provider --cluster <cluster-name> --aprove --region <region-name>
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
Note:Starting with EKS 1.30, the EBS CSI Driver use a default StorageClass object configured using Amazon EBS GP3 volume type. Run the following command to confirm:
Note:No, you cannot use ReadWriteMany for Amazon Elastic Block Store (EBS) volumes in Kubernetes. EBS volumes only support the ReadWriteOnce access mode. This means that they can be mounted as read-write by a single node at a time.

kubectl get storageclass
NAME                           PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
ebs-csi-default-sc (default)   ebs.csi.aws.com         Delete          WaitForFirstConsumer   true                   96s

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
allowVolumeExpansion: true

kubectl apply -f ebs-storageclass.yaml
```

