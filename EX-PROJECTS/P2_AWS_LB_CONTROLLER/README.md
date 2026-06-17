# SETTING UP BOOTSTRAP SERVER
ubuntu 24.04, t2.medium <br>
Pre-requiste: <br>
1.AWS CLI <br>
2.KUBECTL <br>
3.EKSCTL <br>
4.HELM <br>

Update the Server
```bash
sudo apt update && sudo apt upgrade -y
```
### 1.Install AWS CLI
Download AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
Install unzip
```bash
sudo apt install unzip -y
```
Extract and Install
```bash
unzip awscliv2.zip
sudo ./aws/install
```
Verify
```bash
aws --version
```
### 2.Install kubectl
Download Latest Version
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
Make Executable
```bash
chmod +x kubectl
```
Move to PATH
```bash
sudo mv kubectl /usr/local/bin/
```
Verify
```bash
kubectl version --client
```
### 3.Install eksctl
Download eksctl
```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_${PLATFORM}.tar.gz"
```
Extract
```bash
tar -xzf eksctl_${PLATFORM}.tar.gz
```
Move to PATH
```bash
sudo mv eksctl /usr/local/bin
```
Verify
```bash
eksctl version
```
### 4.Install Helm
Download Helm Install Script
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
Give Execute Permission
```bash
chmod 700 get_helm.sh
```
Install
```bash
sudo ./get_helm.sh
```
Verify
```bash
helm version
```
### 5.Configure AWS CLI
```bash
aws configure
```
Provide: <br>

AWS Access Key ID  <br>
AWS Secret Access Key  <br>
Region (e.g. ap-south-1)  <br>
Output format (json)  <br>

Verify:
```bash 
aws sts get-caller-identity
```
# CLUSTER SETUP <br>
### create cluster
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
```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name <cluster-name>
```

get the cluster
```bash
eksctl get cluster
```

IMP:If you created the EKS cluster using eksctl, delete it with:
```bash
eksctl delete cluster --name <cluster-name> --region ap-south-1
```
# AWS LOAD BALANCER CONTEOLLER SETUP.
### 1.Associate IAM OIDC Provider 
```bash
eksctl utils associate-iam-oidc-provider \
  --region ap-south-1 \
  --cluster <cluster-name> \
  --approve
```

### 2.Create Role
Download the Official IAM Policy
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```
Go to you aws ui and create ploicy (create using above link) <br>
POLICY name: AWSLoadBalancerControllerIAMPolicy <br>

Get your OIDC URL
```bash
aws eks describe-cluster \
  --name <cluster-name> \
  --query "cluster.identity.oidc.issuer" \
  --output text
```
Example output:
```bash
https://oidc.eks.ap-south-1.amazonaws.com/id/ABCD123456789
```

Get your AWS Account ID
```bash
aws sts get-caller-identity --query Account --output text
```
Example:
```bash
123456789012
```

### 4.create ROLE:

choose custom policy and paste 
```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::XXXXXXXXXXXXX:oidc-provider/oidc.eks.ap-south-1.amazonaws.com/id/XXXXXXXXXXXXXXXXXXXXXXXXX"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-south-1.amazonaws.com/id/XXXXXXXXXXXXXXXXXXXXXC:sub": "system:serviceaccount:kube-system:aws-load-balancer-controller",
          "oidc.eks.ap-south-1.amazonaws.com/id/XXXXXXXXXXXXXXXXXXXXXXC:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
```

IMPORTANT: attach policy: AWSLoadBalancerControllerIAMPolicy
<br>
give ROLE name: AmazonEKSLoadBalancerControllerRole




### 5.Create Service Account (IRSA)

Replace:

<cluster-name> with your EKS cluster name <br>
<account-id> with your AWS account ID
```bash
eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-role-arn arn:aws:iam::<account-id>:role/AmazonEKSLoadBalancerControllerRole \
  --approve
```

Add the EKS Helm Repository
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```


Get your VPC ID:
```bash
aws eks describe-cluster \
  --name <cluster-name> \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text
```

Install the controller:
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=<vpc-id>
```

Verify Installation
```bash
kubectl get pods -n kube-system | grep aws-load-balancer-controller
```
Expected output:
```bash
aws-load-balancer-controller-xxxxx   1/1 Running
aws-load-balancer-controller-yyyyy   1/1 Running
```

apply 3 files home-deploy.yml, offer.yml,payment.yml

and finally create ingress.yml <br>
this creates alb, listerner rules, target group inside aws.<br>
```bash
kubectl apply -f ingress.yml
```

delete ingress then it delete alb, listerner rules, target groups. 
```bash
kubectl delete ingress microservice-ingress
```

##### OIDC (OpenID Connect)
Definition: <br>
OIDC is a mechanism that allows an EKS pod to securely assume an AWS IAM role without storing AWS access keys inside the pod.

##### ServiceAccount
Definition: <br>
A ServiceAccount is an identity for applications (pods) running inside Kubernetes.
