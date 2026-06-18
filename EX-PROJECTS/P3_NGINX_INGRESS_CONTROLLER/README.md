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
# NGINX INGRESS CONTEOLLER SETUP.
### 1.Add Helm Repository
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
### 2. Install NGINX Ingress Controller
```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer
```
### 3. Verify Pods
```bash
kubectl get pods -n ingress-nginx
```
Wait until all pods are Running.

### 4. Verify Service
```bash
kubectl get svc -n ingress-nginx
```
Expected:
```bash
NAME                                 TYPE           EXTERNAL-IP
ingress-nginx-controller             LoadBalancer   http://aab3d99fbd3904d2f8b81b7e80e80255-1271882760.ap-south-1.elb.amazonaws.com/
```
