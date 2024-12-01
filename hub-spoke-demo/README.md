## Multi-Cluster Deployment with ArgoCD Using AWS-EKS (Hub-Spoke Architecture):

1. **Launch an EC2 Ubuntu Instance**
        - Configure Security Group: Allow ports: SSH (22), HTTP (80), HTTPS (443), 8080 for ArgoCD, Custom TCP Rule (30000–32767) for NodePort services.
        - IAM Role for EC2 with required permissions for EKS and ArgoCD.

2. **SSH into your instance and update it**
```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
sudo apt update && sudo apt upgrade -y
```

3. **Install and configure Aws CLI:**
```bash
# Install awscli
sudo apt install awscli curl unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws --version

# Configure awscli (Obtain ```Access-Key-ID``` and ```Secret-Access-Key``` from the AWS Management Console).
aws configure
```

4. **Install Kubectl:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

5. **Install eksctl**
```bash
curl -s --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

6. **Create and Configure EKS Clusters (one for the hub and two for spokes):**
```bash
eksctl create cluster --name hub-cluster --region us-west-1
eksctl create cluster --name spoke-cluster-1 --region us-west-1
eksctl create cluster --name spoke-cluster-2 --region us-west-1

# Verify clusters
kubectl config get-contexts
kubectl config get-contexts | grep us-west-1

# Switch to the hub cluster
kubectl config use-context arn:aws:eks:us-west-1:<hub-cluster-context>
kubectl config current-context 
```

7. **Install and Configure ArgoCD on the Hub Cluster:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Check if the ArgoCD pods are running.
kubectl get pods -n argocd

# Lists all ConfigMaps in the ```argocd``` namespace and add/edit this (data: server.insecure: "true") in the configmap file. It disables HTTPS enforcement on the ArgoCD server.
kubectl get cm -n argocd 
kubectl edit configmap argocd-cmd-params-cm -n argocd
kubectl describe deployment/argocd-server -n argocd
kubectl rollout restart deployment/argocd-server -n argocd

# Expose ArgoCD Server
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

# Get the external IP of ArgoCD
kubectl get svc -n argocd
kubectl get svc argocd-server -n argocd

# Retrieve the admin password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

# Access ArgoCD UI at http://<external-ip-address:port-address>
```

8. **Install and configure ArgoCD CLI access:**
```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd

# Login using the CLI:
# The ```--insecure``` flag is used to bypass SSL verification if you're using a self-signed certificate or accessing it over HTTP locally.
argocd login  <external-ip-address:port-address> --username admin --password <your-password> --insecure
```

9. **Connect Spoke Clusters to the Hub Cluster:**
```bash
kubectl config get-contexts | grep us-west-1
argocd cluster add <spoke-1-context> --server <external-ip-address:port-address>
argocd cluster add <spoke-2-context> --server <external-ip-address:port-address>

# Verify added clusters:
argocd cluster list
```

10. **Create and deploy applications using ArgoCD:**
```bash
# spoke1-demo-app
argocd app create spoke1-demo-app \
    --repo https://github.com/aqeeladil/argocd-fundam.git \
    --path hub-spoke-demo/guest-book \
    --dest-server <spoke_1_cluster_url> \
    --dest-namespace default
    --sync-policy automated

# spoke2-demo-app
argocd app create spoke2-demo-app \
    --repo https://github.com/aqeeladil/argocd-fundam.git \
    --path hub-spoke-demo/guest-book \
    --dest-server <spoke_2_cluster_url> \
    --dest-namespace default
    --sync-policy automated
```

11. **Verify that the applications are deployed successfully in spoke clusters:**
```bash
kubectl get pods -n default --context <spoke-1-context>
kubectl get configmap -n default --context <spoke-1-context>
kubectl get all -n default --context <spoke-1-context>

kubectl get pods -n default --context <spoke-2-context>
kubectl get configmap -n default --context <spoke-2-context>
kubectl get all -n default --context <spoke-2-context>
```

12. **Delete the Clusters:**
```bash
# Verify the Clusters
eksctl get cluster

eksctl delete cluster --name hub-cluster --region us-west-1
eksctl delete cluster --name spoke-cluster-1 --region us-west-1
eksctl delete cluster --name spoke-cluster-2 --region us-west-1

# Verify Deletion
eksctl get cluster
```