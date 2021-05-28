# demo-clu-flux
Flux CD repo for demo purposes

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
