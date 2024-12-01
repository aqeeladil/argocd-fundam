## Deploying and Managing Kubernetes Applications with Argo CD on a Minikube Cluster using Aws-Ec2 Instance (Standalone Architecture):

1. **Launch an EC2 Ubuntu Instance**
        - Instance Type: Use a t2.medium or larger (Minikube requires at least 2 CPUs and 2GB RAM).
        - Configure Security Group: Allow ports: SSH (22), HTTP (80), HTTPS (443), 8080 for ArgoCD, Custom TCP Rule (30000–32767) for Minikube NodePort services.

2. **SSH into your instance**
```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

3. **Install Required Tools**
```bash
# Update and install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl apt-transport-https

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

4. **Start Minikube with the none driver (because we're using EC2).**
```bash
minikube start --driver=none

# Check if Minikube is running.
minikube status
kubectl get nodes
```

5. **Install Argo CD Using Plain Manifests**
```bash
# Install Argo CD in the argocd namespace
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Check if the ArgoCD pods are running.
kubectl get pods -n argocd
```

6. **Access Argo CD API Server Through Browser**
```bash
kubectl get svc -n argocd

# By default, the Argo CD API server uses a ```ClusterIP``` service. Change (type: ```ClusterIP```) to (type: ```NodePort```). 
kubectl edit svc argocd-server -n argocd

# Get the external IP of ArgoCD
# The output should show the TYPE as NodePort and a PORT(S) value including the assigned port 
kubectl get svc argocd-server -n argocd

minikube service argocd-server -n argocd
minikube ip
```
Access ArgoCD UI at http://<external-ip>.
```bash
# Log In to Argo CD. The default username is admin.
# Retrieve the default password

kubectl get secret -n argocd
kubectl get secret argocd-initial-admin-secret -n argocd
kubectl edit secret argocd-initial-admin-secret -n argocd
echo <secret-password> | base64 --decode
```
*Important Note:*
```yaml
# NodePort makes a service accessible from outside the cluster (web brower) without requiring an Ingress or a LoadBalancer. It maps a port on every node in the cluster to the service, enabling external clients to connect to the service using the node's IP and the assigned port. 
# When you set a service type to NodePort, Kubernetes assigns a port from the range 30000–32767 (default configurable range) on each cluster node. This port is mapped to the target port of the pods associated with the service.
# In production environments, NodePort is typically a building block rather than a standalone solution, often used in conjunction with more robust service types like LoadBalancer or Ingress.

type: NodePort
ports:
  - port: 80       # The service's port
    targetPort: 8080  # The container's port
    nodePort: 30001   # The port exposed on all cluster nodes

# External traffic sent to any node's IP address on port 30001 is forwarded to the service's port (80).
# The service then forwards the request to the pods' targetPort (8080).
```

7. **Create an Application in Argo CD**
Log in to the Argo CD UI. Go to Applications > New Application.
```bash
App Name: demo-app
Project: default
Sync Policy: Automatic
Repository URL: https://github.com/argoproj/argocd-example-apps
Path: guestbook
Cluster URL: Leave as default for Minikube.
Namespace: default
```
```bash
# Verify the Application

kubectl get application -n argocd
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get all -n default

# Watch live updates of deployment:
kubectl get deploy -n argocd
```

___________________________________________________________________________________

**NOTE: (Alternate for steps 6 & 7):**
8. **Access ArgoCD API Server Through CLI and Create an Application:**
```bash
# By default, ArgoCD runs internally. You can expose it using a LoadBalancer or port-forwarding. 
# Port forward the ArgoCD server to access it locally.
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Download and install the ArgoCD CLI:
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
argocd version

# Retrieve the initial admin password:
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d; echo

# Login using the CLI:
# The ```--insecure``` flag is used to bypass SSL verification if you're using a self-signed certificate or accessing it over HTTP locally.
argocd login  <EC2 Public IP>:8080 --username admin --password <your-password> --insecure

# Verify the Login
# This should display a list of ArgoCD applications. 
argocd app list

# Create an application:
argocd app create demo-app \
    --repo https://github.com/argoproj/argocd-example-apps.git \
    --path guestbook \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace default
    --sync-policy automated

# Check the status of the application:
argocd app list
argocd app get demo-app

# Synchronize the Application (If you didn't enable ```automated``` sync while creating the application)
argocd app sync sample-app

# Check the sync and health status:
argocd app wait sample-app


# Verify the application (pods and services):
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl get all -n default

# Watch live updates of deployment:
kubectl get deploy -n argocd

# Troubleshooting
argocd app logs sample-app
```