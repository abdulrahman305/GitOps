# GitOps Starter Kit

This repository contains a basic setup for a GitOps workflow using GitHub Actions and Flux CD.

## Next Steps

All the necessary files have been generated. Here’s how to get your GitOps system running:

### 1. Create a GitHub Repository

- Go to your GitHub account and create a new, empty repository. For example, name it `my-gitops-repo`.

### 2. Push These Files

- Navigate to this `gitops-starter` directory in your terminal and run the following commands:

```bash
# Replace YOUR_USERNAME and YOUR_REPO_NAME with your actual GitHub details
git init
git add .
git commit -m "Initial GitOps setup"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git push -u origin main
```

### 3. Update the Deployment Image

- Open the `kubernetes/deployment.yaml` file.
- Find the `image` field and **update it** to point to your new repository. For example:

```yaml
image: ghcr.io/abdulrahman305/my-gitops-repo:latest
```

- Commit and push this change.

### 4. Set Up Your Kubernetes Cluster & Flux

- **Prerequisites:**
  - You need a Kubernetes cluster. For local testing, you can use [minikube](https://minikube.sigs.k8s.io/docs/start/).
  - You need the `kubectl` command-line tool installed and configured to talk to your cluster.
  - You need to install the [Flux CLI](https://fluxcd.io/flux/installation/).

- **Bootstrap Flux:**

Run the following command, replacing the values with your GitHub information. This command tells Flux to monitor your repository and apply the configurations found in the `kubernetes/` path.

```bash
flux bootstrap github \
  --owner=YOUR_USERNAME \
  --repository=YOUR_REPO_NAME \
  --branch=main \
  --path=./kubernetes \
  --personal
```

### 5. Observe the Magic!

- Once you run the bootstrap command, Flux will be installed on your cluster.
- It will see the `deployment.yaml` and `service.yaml` files in your repository.
- It will apply them to your cluster, and Kubernetes will pull the `latest` image that your GitHub Action built.
- You can check the status with `kubectl get deployments` and `kubectl get services`.

From now on, any change you push to the `kubernetes/` directory will be automatically applied to your cluster by Flux.

## Autonomous Operations

This starter kit is now configured with several autonomous capabilities:

- **Self-Healing:** The application has health checks (`livenessProbe`). If it becomes unresponsive, Kubernetes will automatically restart it.
- **Auto-Scaling:** The `HorizontalPodAutoscaler` will automatically increase or decrease the number of application pods (from 2 to 10) based on CPU utilization.
- **Automated Deployments:** When you push a code change to the `app/` directory, the CI/CD pipeline will automatically build a new container image, push it, and update the `deployment.yaml` file with the new image tag. Flux will then automatically roll out this new version to the cluster.

### **IMPORTANT: Auto-Scaling Prerequisite**

For the `HorizontalPodAutoscaler` (HPA) to function, your Kubernetes cluster must have the **Metrics Server** installed. It is not installed by default in many clusters.

- **To check if it's installed:** `kubectl get deployment metrics-server -n kube-system`
- **If it is not installed, you can install it with:** `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

## Level 2: Fully Autonomous Releases with Flagger

This repository is now configured for the highest level of autonomous operations: **Automated Canary Releases with Rollbacks**.

The `Canary` resource tells Flagger to watch our deployment. When a new image is deployed, Flagger will:
1. Gradually shift traffic to the new version.
2. Analyze Prometheus metrics for success rate and latency.
3. If metrics are healthy, it will fully promote the release.
4. **If metrics degrade, it will automatically roll back to the previous stable version.**

### **IMPORTANT: Flagger Prerequisite**

This capability requires **Flagger** and a **compatible Ingress Controller** (like NGINX) to be installed on your cluster.

1. **Install an Ingress Controller:**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

2. **Install Flagger:**
```bash
# Add the Flagger Helm repository
helm repo add flagger https://flagger.app

# Install Flagger for the NGINX Ingress controller
helm upgrade -i flagger flagger/flagger \
--namespace=flagger-system --create-namespace \
--set-string meshProvider=nginx
```

## Level 4: Autonomous Security and Maintenance

To complete the system, we have now integrated autonomous security scanning and dependency management.

- **`codeql.yml`**: This workflow automatically performs deep source code analysis using GitHub CodeQL to find security vulnerabilities (CWEs) and other bugs in your own code before they ever reach production.

- **`dependabot.yml`**: This configuration enables GitHub Dependabot to constantly monitor your third-party dependencies. When a vulnerability is discovered in a package you use, Dependabot will **automatically create a pull request** to upgrade to a patched version.

### The Closed Loop

This creates a fully closed-loop autonomous system. When Dependabot creates a PR for a security patch, our existing `automerge.yml` workflow will take over. If all tests pass, it will **automatically merge the security fix**, which will then be deployed to production via the GitOps workflow. This is a powerful, hands-off way to keep your application secure.
## Level 3: Autonomous Pull Request Management

To complete the autonomous lifecycle, this repository now includes workflows to manage pull requests automatically.

- **`automerge.yml`**: When a pull request is opened, this workflow will check if it has any conflicts and if all tests are passing. If everything is clean, it will **automatically merge the PR**.
- **`sync-branches.yml`**: When a change is merged into the `main` branch, this workflow will find all other open pull requests and **automatically update them** with the latest code from `main`. If it finds a conflict, it will label the PR for manual review.

This system works to keep your repository in a constant state of readiness, achieving a "zero open pull requests" and "fully synced" state by handling all non-conflicting changes autonomously.
