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
argocd login <External Loadbalancer IP>
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

# Deploy the application on the kubenetes cluster
install Nginx controller
```
kubectl create namespace ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml -n ingress-nginx
```
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

Check the rollout
```
kubectl argo rollouts list rollouts
```

Check the rollout status
```
kubectl argo rollouts get rollout realtimeapp
```
change the image to 1.0.0 on the application's deployment
