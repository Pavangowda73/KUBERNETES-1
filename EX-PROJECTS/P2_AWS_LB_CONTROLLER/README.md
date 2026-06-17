# create cluster
```bash
eksctl create cluster \
  --name my-cluster \
  --region ap-south-1 \
  --nodegroup-name workers \
  --node-type t2.medium \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --managed
```


Connect to Existing EKS Cluster

aws eks update-kubeconfig \
  --region ap-south-1 \
  --name <cluster-name>

get the cluster

eksctl get cluster

If you created the EKS cluster using eksctl, delete it with:

eksctl delete cluster --name <cluster-name> --region ap-south-1


Associate IAM OIDC Provider

eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster <cluster-name> \
  --approve


Download the Official IAM Policy

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

create using above link
POLICY name: AWSLoadBalancerControllerIAMPolicy



Get your OIDC URL

aws eks describe-cluster \
  --name <cluster-name> \
  --query "cluster.identity.oidc.issuer" \
  --output text

Example output:

https://oidc.eks.ap-south-1.amazonaws.com/id/ABCD123456789



Get your AWS Account ID

aws sts get-caller-identity --query Account --output text

Example:

123456789012



create ROLE:

choose custom policy and paste 

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::147997119633:oidc-provider/oidc.eks.ap-south-1.amazonaws.com/id/EC00E1BB34CBEC79C0088F02B6E8E92C"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-south-1.amazonaws.com/id/EC00E1BB34CBEC79C0088F02B6E8E92C:sub": "system:serviceaccount:kube-system:aws-load-balancer-controller",
          "oidc.eks.ap-south-1.amazonaws.com/id/EC00E1BB34CBEC79C0088F02B6E8E92C:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}

attach policy: AWSLoadBalancerControllerIAMPolicy

give ROLE name: AmazonEKSLoadBalancerControllerRole




Create Service Account (IRSA)

Replace:

<cluster-name> with your EKS cluster name
<account-id> with your AWS account ID

eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-role-arn arn:aws:iam::<account-id>:role/AmazonEKSLoadBalancerControllerRole \
  --approve


Add the EKS Helm Repository

helm repo add eks https://aws.github.io/eks-charts
helm repo update



Get your VPC ID:

aws eks describe-cluster \
  --name <cluster-name> \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text


Install the controller:

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=<vpc-id>


Verify Installation
kubectl get pods -n kube-system | grep aws-load-balancer-controller

Expected output:

aws-load-balancer-controller-xxxxx   1/1 Running
aws-load-balancer-controller-yyyyy   1/1 Running


apply 3 files home-deploy.yml, offer.yml,payment.yml

and finally ingress.ymal
this creates alb, listerner rules, target group inside aws.

delete ingress then it delete alb, listerner rules, target groups. 








