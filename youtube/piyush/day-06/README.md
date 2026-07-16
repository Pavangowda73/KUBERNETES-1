### Kind Cluster

#### Creating kind cluster on ubuntu 24.04

Pre-requiste: <br>
1.Docker <br>
2.KUBECTL <br>
3.kind <br>

1.Docker
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y

# Allow your current user to run Docker without sudo
sudo usermod -aG docker $USER
```

2.KUBECTL
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client
```

3.Kind
```bash
# Detect your system architecture (AMD64 vs ARM64) and download the binary
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-arm64

# Make it executable and move it to /usr/local/bin
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Verify installation
kind version
```


#### creating kind cluster without worker nodes

```bash
kind create cluster --image kindest/node:v1.30.10 --name cka-cluster-2 
```

check for nodes
```bash
kubectl get nodes
```

#### creating kind cluster with worker nodes

first create config.yaml
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

```bash
kind create cluster --image kindest/node:v1.30.10 --name cka-cluster-2 --config config.yaml
```

check for nodes
```bash
kubectl get nodes
```

check for kind clusters
```bash
kubectl config get-contexts
```

switch between clsuter
```bash
kubectl config use-context <name-of-cluster>
```

eg:  kubectl config use-context kind-cka-cluster-2



