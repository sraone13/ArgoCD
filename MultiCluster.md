## **Multi-Cluster Deployment with GitOps (Argo CD)**

---

## **1. What is Multi-Cluster Deployment?**

Organizations use **multiple Kubernetes clusters** for different environments:

- Dev  
- QA  
- Pre-Prod  
- Prod  

Each environment runs on a **separate Kubernetes cluster**.

For new features, developers may request **new clusters (feature clusters)**.

---

## **2. Traditional CI/CD vs GitOps**

### **Traditional CI/CD (Drawbacks)**

Deployments are done using:

- Shell scripts  
- Python scripts  
- Ansible  

**Problems:**

- No proper tracking of changes  
- Anyone can manually change YAML in the cluster  
- No visibility on who changed what  
- Hard to identify root cause when application fails  
- No automatic rollback  

---

## **3. GitOps Approach**

### **Key Concept**

- Git is the **single source of truth**
- CI pulls application code from Git
- CD pulls Kubernetes manifests from Git

### **GitOps Tools**

- Argo CD  
- Flux  
- Spinnaker  

---

## **4. How Argo CD Works**

- Argo CD is a **Kubernetes Controller**
- It continuously monitors:
  - Git repository
  - Kubernetes cluster state

All Kubernetes manifests must be stored in Git:

- Deployments  
- Services  
- ConfigMaps  
- Secrets  

### **Desired vs Actual State**

If someone manually modifies resources in the cluster:

- Argo CD detects configuration drift  
- Automatically reverts the change  
- Syncs cluster state back to Git  

**Example:**

- Git value: `replicas: 5`  
- Manual change: `replicas: 10`  
- Argo CD reverts it back to `5`

---

## **5. Benefits of Argo CD (GitOps)**

- Tracking  
- Auditing  
- Monitoring  
- Auto-healing  
- Auto-rollback  
- Secure deployments  
- No manual cluster changes  

---

## **6. Argo CD Deployment Models**

### **1️⃣ Hub-Spoke Model (Centralized)**

- One central **Hub cluster**
- Argo CD installed only on the hub
- Hub manages multiple spoke clusters (K1, K2, K3)

**Flow:**

Git → Argo CD (Hub) → Multiple Clusters

yaml
Copy code

---

### **2️⃣ Standalone Model**

- Argo CD installed on each cluster
- Each cluster deploys applications independently

**Flow:**

Git → Argo CD (Dev)
Git → Argo CD (QA)
Git → Argo CD (Stage)

yaml
Copy code

---

## **7. CI Pipeline (Jenkins Example)**

### **CI Stages**

- Build  
- Unit Tests (UT)  
- Static Code Analysis  
- SAST  
- DAST  
- Docker Image Build  
- Trivy Scan  

### **CD**

- Deployment handled by **Argo CD via GitOps**

---

## **8. Architecture Overview**

Git Repository
↓
Argo CD Controller
↓
Kubernetes Cluster(s)

yaml
Copy code

---

## **9. Prerequisites**

- kubectl  
- eksctl  
- AWS CLI  
- Argo CD CLI  
- AWS account access  

---

## **10. Setup kubectl**

### **Download kubectl**

```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.2/2025-11-13/bin/linux/arm64/kubectl
Download checksum
bash
Copy code
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.2/2025-11-13/bin/linux/arm64/kubectl.sha256
Verify checksum
bash
Copy code
sha256sum -c kubectl.sha256
Apply execute permissions
bash
Copy code
chmod +x ./kubectl
Move binary to PATH
bash
Copy code
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
11. Setup eksctl
bash
Copy code
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" \
| grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp
sudo install -m 0755 /tmp/eksctl /usr/local/bin
12. Install AWS CLI
bash
Copy code
sudo apt update && sudo apt upgrade -y
sudo apt install unzip curl -y

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
Verify
bash
Copy code
aws --version
Configure AWS
bash
Copy code
aws configure
13. EKS Cluster Creation (Example)
bash
Copy code
eksctl create cluster --name hub-cluster --region us-west-1
eksctl create cluster --name spoke-cluster-1 --region us-west-1
eksctl create cluster --name spoke-cluster-2 --region us-west-2
Verify Clusters
bash
Copy code
kubectl config get-contexts | grep us-west
14. Switch to Hub Cluster
bash
Copy code
kubectl config use-context hub-cluster
kubectl config current-context
15. Install Argo CD
bash
Copy code
kubectl create namespace argocd

kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Verify
bash
Copy code
kubectl get pods -n argocd
16. Configure Argo CD Server (Insecure Mode)
bash
Copy code
kubectl get cm -n argocd
kubectl edit configmap argocd-cmd-params-cm -n argocd
Add:

yaml
Copy code
data:
  server.insecure: "true"
Verify:

bash
Copy code
kubectl describe deployment argocd-server -n argocd
17. Expose Argo CD UI
bash
Copy code
kubectl get svc -n argocd
kubectl edit svc argocd-server -n argocd
Change:

yaml
Copy code
type: NodePort
Copy NodePort IP and Port

Open in browser

Update EC2 Security Group inbound rules if required

18. Login to Argo CD UI
bash
Copy code
kubectl get secrets -n argocd
kubectl describe secret argocd-initial-admin-secret -n argocd
Decode password:

bash
Copy code
echo <base64-password> | base64 --decode
Login details:

Username: admin

Password: decoded value

19. Add Spoke Clusters to Argo CD
bash
Copy code
argocd login <ARGOCD_IP>:<PORT>
argocd cluster add <spoke-cluster-context>
Repeat for all spoke clusters.

Verify in UI:

Settings → Clusters

Refresh → All clusters visible

20. Deploy Application to Multiple Clusters
Steps
Create application in Argo CD UI

Provide:

Application name

Git repository

Path

Destination cluster

Namespace

Repeat for other clusters

21. Verification
bash
Copy code
kubectl get pods
Update YAML in Git

Argo CD syncs changes automatically (within ~3 minutes)

22. Summary
CI → Git

CD → Argo CD

Git is the single source of truth

No manual cluster changes

Secure, scalable, and auditable deployments
