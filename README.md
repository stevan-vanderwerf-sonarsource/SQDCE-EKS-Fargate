# SQDCE-EKS-Fargate

## AWS Fargate considerations when running on Kubernetes (EKS)
- https://docs.aws.amazon.com/eks/latest/userguide/fargate.html 
- Persistence is available through EFS (EBS not supported for Fargate), potentially lower performance for Elasticsearch
- Dynamic provisioning is not available for EFS when using Fargate, volumes need to be statically provisioned

## Create Amazon EKS Cluster
- https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html 
- You can create a cluster by using `eksctl`
- Make sure to use `–fargate` parameter to provision the cluster with fargate rather than the default Node Groups with EC2
- https://eksctl.io/usage/fargate-support/ 

## (IAM) OpenID Connect (OIDC) provider for cluster
- https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html 
- To use AWS Identity and Access Management (IAM) roles for service accounts, an IAM OIDC provider must exist for your cluster's OIDC issuer URL.

## Amazon EFS CSI driver
- Amazon EFS CSI driver needs to be deployed to the EKS cluster to manage the lifecycle of Amazon EFS file systems
- Create the Amazon EBS CSI driver IAM role: https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html 
- Since Aug 2023 Amazon supports installing the Amazon EFS CSI driver via add-on: https://docs.aws.amazon.com/eks/latest/userguide/managing-add-ons.html#creating-an-add-on 

## Create an Amazon EFS file system for Amazon EKS
- Steps for creating an Amazon EFS file system for Amazon EKS
- https://github.com/kubernetes-sigs/aws-efs-csi-driver/blob/master/docs/efs-create-filesystem.md 
- Test the Cluster integration with EFS by creating a statically provisioned Amazon EFS persistent volume (PV) and accessing it from multiple pods with the ReadWriteMany (RWX) access mode. 
- https://github.com/kubernetes-sigs/aws-efs-csi-driver/tree/master/examples/kubernetes/multiple_pods#multiple-pods-read-write-many 

## Create persistent storage for SonarQube
- Create PV, PVC, and Storage Class
- https://aws.amazon.com/blogs/storage/persistent-storage-for-kubernetes/ (Static provisioning using Amazon EFS section section)
- A separate PV/PVC should be created for each Search Node name:
_sonarqube-sonarqube-dce-sonarqube-sonarqube-dce-search-0
Sonarqube-sonarqube-dce-sonarqube-sonarqube-dce-search-1
sonarqube-sonarqube-dce-sonarqube-sonarqube-dce-search-2_
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sonarqube-sonarqube-dce-sonarqube-sonarqube-dce-search-0
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  resources:
    requests:
      storage: 5Gi
```
Make sure PV and PVC shows Status as ‘Bound’

## Configure helm chart values.yaml
- https://github.com/SonarSource/helm-chart-sonarqube/blob/master/charts/sonarqube-dce/values.yaml 
- Add `existingClaim` key
```
  persistence:
    enabled: true
    ## Set annotations on pvc
    annotations: {}

    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    type: pvc
    storageClass:
    existingClaim: sonarqube-sonarqube-dce-sonarqube-sonarqube-dce-search
    accessMode: ReadWriteOnce
    size: 5Gi
    uid: 1000
```
## Deploy SonarQube DCE helm chart from file
- https://github.com/SonarSource/helm-chart-sonarqube/tree/master/charts/sonarqube-dce

```helm upgrade --install -f custom-values.yaml sonarqube --set ApplicationNodes.jwtSecret=$JWT_SECRET sonarqube/sonarqube-dce```



