1️⃣ What is GitOps?

GitOps is a practice that uses Git as the single source of truth to deliver and manage applications and infrastructure.

Key Points

All changes are made only through Git

Git provides:

Versioning

Change history

Traceability

The system automatically syncs the desired state from Git to the runtime environment (e.g., Kubernetes)

2️⃣ Why GitOps?
Problems Without GitOps

Code changes in Git are tracked

Changes made directly in Kubernetes are not tracked

CI pipelines provide tracking, but traditional CD does not

How GitOps Helps

Ensures all deployments happen via Git

Enforces consistency between Git and the cluster

Can be used for application deployment and infrastructure management

3️⃣ GitOps Workflow (High Level)

DevOps Engineer updates Kubernetes YAML manifests in GitHub

A Pull Request (PR) is created

Team member reviews and approves the PR

GitOps tool (Argo CD) detects changes in Git

Argo CD deploys changes to the Kubernetes cluster

Argo CD continuously ensures:

Git state == Cluster state

4️⃣ GitOps Principles
1. Declarative

Desired state is defined declaratively (YAML)

Kubernetes manifests describe what the system should look like

2. Versioned and Immutable

Desired state is stored in Git

Git enforces:

Versioning

Immutability

Complete change history

3. Pulled Automatically

GitOps agents (e.g., Argo CD) pull changes from Git

No manual push to Kubernetes

4. Continuously Reconciled

GitOps agents continuously compare:

Desired State (Git)

Actual State (Cluster)

Any drift is automatically corrected

5️⃣ Is GitOps Only for Kubernetes?

❌ No. GitOps is a general principle.

✅ However, most popular GitOps tools are focused on Kubernetes.

6️⃣ Advantages of GitOps

Improved security

Full version history (audit trail)

Automatic upgrades

Auto-healing of unauthorized changes

Continuous reconciliation

7️⃣ Popular GitOps Tools
Tool	GitOps Support
Argo CD	✅ GitOps-native
Flux	✅ GitOps-native
Jenkins	❌ Not GitOps-native
Spinnaker	⚠️ Partial GitOps support
8️⃣ GitOps in a Nutshell
Git  <---- sync ----  Argo CD  ---- sync ---->  Kubernetes


Argo CD keeps GitHub YAML files and Kubernetes in sync

Any change in Git is automatically deployed

Desired state is always enforced

9️⃣ Argo CD Architecture
Core Components
1. API Server

Used by UI and CLI

Handles authentication and authorization

2. Dex

Provides SSO support

Supports OIDC, LDAP, etc.

3. Repo Server

Connects to Git repositories

Fetches manifests and desired state

4. Application Controller

Connects to Kubernetes

Compares desired vs actual state

Applies changes if required

5. Redis

Used for caching application data

🔟 Architecture Summary

Repo Server → Talks to Git

Application Controller → Talks to Kubernetes

Both continuously compare and keep the cluster in sync

🔧 Argo CD Installation (Basic)
Step 1: Create Namespace
kubectl create namespace argocd

Step 2: Install Argo CD
kubectl apply -n argocd -f \
https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Step 3: Verify Pods
kubectl get pods -n argocd -w

Step 4: Check Services
kubectl get svc -n argocd


You will see:

argocd-server

🌐 Expose Argo CD UI
Step 5: Change Service Type to NodePort
kubectl edit svc argocd-server -n argocd


Change:

type: NodePort

Step 6: Access Argo CD UI
kubectl get svc argocd-server -n argocd


OR use port-forward:

kubectl port-forward svc/argocd-server -n argocd 8080:443


Open browser:

https://<Node-IP>:<NodePort>
OR
https://localhost:8080

🔐 Get Argo CD Admin Password
Default Credentials

Username: admin

Get Secret
kubectl get secrets -n argocd
kubectl describe secret argocd-initial-admin-secret -n argocd

Decode Password
echo <password> | base64 --decode

📦 Example Applications for Practice

Official Argo CD examples:

https://github.com/argoproj/argocd-example-apps

🚀 Creating an Application in Argo CD (UI)

Click Create Application

Enter:

Application Name

Project Name

Sync Policy → Automatic

Repository URL → GitHub repo

Revision → HEAD

Path → Folder containing YAML (e.g., guestbook)

Cluster URL → Auto-populated

Namespace → default

Click Create

✅ Deployment Verification
kubectl get deploy


✔️ Confirms application is deployed via Argo CD
