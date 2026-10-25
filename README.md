# Argo CD + Minikube GitOps demo

A tiny NGINX deployment managed by Argo CD from a GitHub repository.

## 1. Start Minikube

```bash
minikube start
kubectl get nodes
```

## 2. Install Argo CD in Minikube

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd rollout status deployment/argocd-server
```

Get the initial admin password (if the initial secret still exists):

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath='{.data.password}' | base64 -d; echo
```

Open the Argo CD UI in a terminal and keep it running:

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

Visit https://localhost:8080 and sign in as `admin`.

## 3. Push these files to GitHub

Create a repository, for example `argocd-minikube-demo`, then from this folder:

```bash
git init
git add .
git commit -m "Add demo Kubernetes manifests"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/argocd-minikube-demo.git
git push -u origin main
```

Replace `YOUR_GITHUB_USERNAME` with your GitHub username. Do not commit passwords or tokens.

## 4. Create the Argo CD application

Replace the repository URL in `argocd/application.yaml` with your own public GitHub repository URL, then apply it:

```bash
kubectl apply -f argocd/application.yaml
```

This uses the in-cluster Argo CD API and creates an Application that watches the `k8s/` directory on the `main` branch. For a private GitHub repository, add repository credentials to Argo CD first.

Check status:

```bash
kubectl get applications -n argocd
kubectl get pods -n demo
kubectl get svc -n demo
```

## 5. Open the sample app

```bash
kubectl -n demo port-forward svc/nginx 8081:80
```

Visit http://localhost:8081.

## Make a change

Edit `k8s/deployment.yaml` (for example, change `replicas: 2` to `replicas: 3`), commit and push. Argo CD will detect the Git change and sync it automatically because `automated` sync is enabled.

## Notes

- This demo uses a public repository URL by default. If your repository is private, configure repository access in Argo CD; never put a GitHub token in this YAML.
- The namespace `demo` is created by `k8s/namespace.yaml`.
- For a disposable local learning environment only; adapt resource limits, policies, and access controls before production use.
