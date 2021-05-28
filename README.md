# demo-clu-flux
Flux CD repo for demo purposes

# Install ArgoCD

Login to the GCP cluster
```
gcloud container clusters get-credentials $(hostname) --zone $(gcloud compute instances list $(hostname) --format "value(zone)") --project devops-course-architecture
```
install argoCD CLI
```
brew install argocd
```
Install argoCD on kubernetes
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Create **LoadBalancer** type services
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
Wait until the **LoadBalancer** service's external IP is granted 
```
kubectl get svc -n argocd --wait | grep -i loadbalancer
```
Getting the inital password
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Login to argocd
```
argocd login  <External Loadbalancer IP> --insecure --username admin --password <Inital Password> 
```
Change the password using the command:
```
argocd account update-password
```

List all clusters contexts in your current kubeconfig and get the cluster name
```
kubectl config get-contexts -o name
```
Add the cluster as a deployment cluster
```
argocd cluster add <Cluster Name>
```

# Install ArgoCD rollout

Install the rollout operator
```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml
```
Install kubectl rollouts
```
brew install argoproj/tap/kubectl-argo-rollouts
```

# Deploy the application on the kubenetes cluster
Enter from the browser to the ArgoCD  UI
```
http://<External Loadbalancer IP>
```
install Nginx controller
```
kubectl create namespace ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml -n ingress-nginx
```
Deploy the application from this file in the UI
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sdp-app
spec:
  destination:
    name: ''
    namespace: default
    server: 'https://kubernetes.default.svc'
  source:
    path: rollouts/realtime-can
    repoURL: 'https://github.com/ormam/demo-clu-flux.git'
    targetRevision: HEAD
  project: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

## Sync the application

Check the rollout
```
kubectl argo rollouts list rollouts
```
Get the ingress's IP
```
k get ingress -A
```
## Browse to the website with the ingress's IP

## Change the image to 1.0.0 on the application's deployment

Check the rollout status progressing until step 5
```
kubectl argo rollouts get rollout realtimeapp -w
```

## Resume the cannary rollout from the UI
