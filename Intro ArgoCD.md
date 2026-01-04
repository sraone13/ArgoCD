1.	What is GitOps?
GitOps is a practice that uses Git as a single source of truth to deliver and manage applications and infrastructure.
	All changes are made through Git.
	Git maintains history, tracking, and versioning.
	The system automatically syncs the desired state from Git to the runtime environment (like Kubernetes).




2.	Why GitOps?

	When we commit code to Git, changes are tracked.
	But when changes are made directly in Kubernetes, there is no tracking.
	CI pipelines have tracking, but traditional CD does not.
	GitOps solves this by ensuring all deployments happen via Git.
	👉 GitOps is also used for infrastructure management.



3.	GitOps Workflow (High Level)

	DevOps Engineer updates Kubernetes YAML manifests in GitHub.
	A Pull Request is created.
	Team member reviews and approves the PR.
	GitOps tool (Argo CD) detects changes in Git.
	Argo CD deploys the changes to the Kubernetes cluster.
	Argo CD continuously ensures Git state == cluster state.



4.	GitOps Principles
1. Declarative:
The system’s desired state must be described declaratively (YAML).
Kubernetes manifests define what the system should look like.

2. Versioned and Immutable
Desired state is stored in Git.
Git enforces:
Versioning
Immutability
Complete change history

3. Pulled Automatically
Software agents (like Argo CD) pull changes from Git.
No manual push to Kubernetes.

4. Continuously Reconciled
Agents continuously compare:
Desired state (Git)
Actual state (Cluster)
If drift is detected, it is automatically corrected.



5.	Is GitOps Only for Kubernetes?

No, GitOps is a general principle.




6.	However, popular GitOps tools like Argo CD and Flux mainly target Kubernetes.

Advantages of GitOps
Security
Full version history (audit trail
Automatic upgrades
Auto-healing of unwanted changes
Continuous reconciliation




7.	Popular GitOps Tools:

Argo CD
Flux
Jenkins ❌ (Not GitOps-native)
Spinnaker (supports GitOps patterns but not pure GitOps)




8.	GitOps in a Nutshell:
Git  <----sync----  Argo CD  ----sync---->  Kubernetes


Argo CD maintains sync between GitHub YAML files and Kubernetes.
Any change in Git is automatically deployed.
Desired state is always enforced.

9.	Argo CD Architecture

Core Components
API Server: Used by UI and CLI and Handles authentication and authorization
Dex: Adds SSO capabilities (OIDC, LDAP, etc.)
Repo Server: Connects to Git repositories and Fetches manifests and desired state
Application Controller: Connects to Kubernetes and Compares actual vs desired state and Applies changes if needed
Redis: Used for caching application data

10.	Architecture Summary

Repo Server talks to Git.
Application Controller talks to Kubernetes.
Both compare states and keep the cluster in sync.


Argo CD Installation (Basic)

Step 1: Create Namespace
kubectl create namespace argocd

Step 2: Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Step 3: Verify Pods
kubectl get pods -n argocd -w

Step 4: Check Services
kubectl get svc -n argocd

You will see argocd-server.

Step 5: Expose Argo CD UI
By default, Argo CD runs as ClusterIP.

Edit service:
kubectl edit svc argocd-server -n argocd

Change:
type: NodePort

Step 6: Access Argo CD UI
kubectl get svc argocd-server -n argocd
kubectl port-forward svc/argocd-server -n argocd 8080:443


Copy Node IP or Server Ip address with Port
Open in browser using HTTP/HTTPS

Step 7: Get Admin Password
Username: admin
Password: 
kubectl get secrets -n argocd
kubectl describe secret argocd-initial-admin-secret -n argocd


Decode password:
echo <password> | base64 --decode

Example Applications for Practice
Official Argo CD examples:
https://github.com/argoproj/argocd-example-apps

Creating an Application in Argo CD (UI)

Click Create Application
Enter:
Application Name
Project Name
Sync Policy → Automatic
Repository URL → GitHub repo URL
Revision → HEAD
Path → folder containing YAML (example: guestbook)
Cluster URL → Auto-populated
Namespace → default
Click Create

Deployment Verification
kubectl get deploy

Confirms application is deployed via Argo CD.
