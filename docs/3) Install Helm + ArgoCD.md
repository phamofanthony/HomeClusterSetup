# Overview
This doc outlines setting up Helm on the control plane node and ArgoCD to manage applications.

### Install Helm (control plane)
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Install ArgoCD
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd --create-namespace \
  --set server.service.type=LoadBalancer
  ```
Wait for it to finish setting up. Once done, all the ArgoCD pods should be in running state. 
```bash
kubectl -n argocd get pods
```

###