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

### Access the ArgoCD web UI
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

kubectl port-forward -n argocd svc/argocd-server 8080:443
```
Login with the username admin and the password from above.

### Create an ArgoCD instance for the app of apps
This will manage all other apps in the repository. In clusters/mk1, create app-of-apps-yaml. Copy the one from this repository.

Apply it with the following command:
```bash
kubectl apply -f clusters/main/app-of-apps.yaml
```

Now, we can create subfolders in clusters/mk1/base/, and this app-of-apps project will automatically manage them for us.