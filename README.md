🚀 PROJECT TITLE:
Solve Real-Time Problem: Enforce Kubernetes Security with Kyverno | Real-Time #Kubernetes Project
💡 Project Background

Yes — I faced a real-time problem while working with Kubernetes (K8s).
The challenge was to enforce security, governance, and compliance automatically across multiple clusters — without manually writing and maintaining custom admission controllers.

So, I built a real-time solution by myself using Kyverno and ArgoCD, inspired by the Kubernetes explanation of Abhishek Veermalla.

This project is called:

“Enforce Automated Kubernetes Cluster Security using Kyverno Policy Generator and ArgoCD”

It works seamlessly on AWS, Azure, GCP, or on-premises clusters — without any extra configuration.
<img width="900" height="360" alt="Screenshot 2025-11-06 203634" src="https://github.com/user-attachments/assets/eb42cd10-279f-47e0-afa1-91291c1a3879" />

<img width="764" height="459" alt="Screenshot 2025-11-06 203712" src="https://github.com/user-attachments/assets/4794ab24-9cf4-4f41-a0a3-727ead560c3b" />

🧩 What You’ll Learn

With this project setup, you can:
1️⃣ Generate → Automatically create a default NetworkPolicy when a new namespace is created.
2️⃣ Validate → Block users from using the :latest tag in deployments or pods.
3️⃣ Mutate → Automatically attach Pod Security Policies to pods without security configurations.
4️⃣ Verify Images → Check if images are properly signed and verified before allowing deployment.

🏗️ High-Level Design

The DevOps Engineer writes Kyverno Policy YAML files and commits them to a Git repository.

ArgoCD, configured with auto-sync, monitors that repo.

Whenever a policy is updated, ArgoCD automatically deploys it to the Kubernetes cluster.

Kyverno enforces, validates, or mutates resources based on the defined rules.

⚙️ Step-by-Step Project Setup
Step 1: Prerequisites

Make sure you have:

A running Kubernetes cluster (EKS, AKS, GKE, or Minikube)

kubectl, helm, and git installed

Cluster admin access

Step 2: Install Kyverno

Kyverno is a Kubernetes-native policy engine that helps enforce security and compliance declaratively.

🧭 Option 1 — Using Helm
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update

# Install Kyverno in HA mode
helm install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=3

🧩 Option 2 — Using Manifest
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.8.5/install.yaml


✅ Verify installation:

kubectl get pods -n kyverno

Step 3: Install ArgoCD

ArgoCD will handle GitOps automation, deploying your Kyverno policies directly from your Git repo.

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/install.yaml


Access the ArgoCD dashboard:

kubectl port-forward svc/argocd-server -n argocd 8080:443


Then open 👉 https://localhost:8080

Step 4: Create a Kyverno Policy

Example: Enforce CPU and Memory limits for every container.

apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-requests-limits
spec:
  validationFailureAction: enforce
  background: true
  rules:
  - name: validate-requests-limits
    match:
      resources:
        kinds:
        - Pod
        - Deployment
    validate:
      message: "Each container must have CPU/memory requests and limits."
      anyPattern:
      - spec:
          containers:
          - resources:
              requests:
                cpu: "?*"
                memory: "?*"
              limits:
                cpu: "?*"
                memory: "?*"


Apply the policy:

kubectl apply -f require-requests-limits.yaml

Step 5: Test the Policy

Try deploying a non-compliant deployment:

kubectl create ns demo
kubectl -n demo create deploy bad-deploy --image=nginx


Expected output:

Error from server: admission webhook "validate.kyverno.svc" denied the request:
require-requests-limits failed: Each container must have CPU/memory requests and limits.


✅ The pod was blocked — policy enforcement successful!

Step 6: Integrate with ArgoCD

1️⃣ Push your Kyverno policies to a GitHub repo.
2️⃣ In ArgoCD, create a new Application pointing to that repo.
3️⃣ Enable auto-sync to keep your cluster in continuous compliance.

🧠 How It Works (Architecture Flow)

kubectl apply or ArgoCD syncs → creates resource.

Kyverno webhook intercepts request → checks against ClusterPolicy.

Based on rules:

Validate → Block or allow resource.

Mutate → Add missing configurations.

Generate → Create additional resources.

VerifyImages → Check image authenticity.

Kyverno returns admit/deny to the Kubernetes API.

🧩 Practical Takeaways

✅ Kyverno = Policy-as-Code engine for Kubernetes (no coding needed).
✅ ArgoCD = Continuous GitOps deployment of policies.
✅ audit = Log & report violations, allow resource.
✅ enforce = Block non-compliant resource creation.
✅ Works across AWS / Azure / GCP / On-Prem with no extra setup.

