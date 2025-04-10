https://www.youtube.com/watch?v=cqWa_ruMW3w
https://www.eksworkshop.com/docs/fundamentals/storage/efs/efs-csi-driver
https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html
```
i)Install EKSCTL
ii)Check IAM OIDC provider is associated with cluster
oidc_id=$(aws eks describe cluster --name <cluster-name> --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
aws iam list-open-id-connect-providers | grep $(oidc_id)
#If o/p is returned then we already have OIDC provider for cluster then no need to run the below command
eksctl utils associate-oidc-provider --cluster <cluster-name> --aprove --region <region-name>
```
