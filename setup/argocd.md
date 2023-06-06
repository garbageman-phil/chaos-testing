```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

be sure to restart minikube-tunnel 

add the cluster
`argocd cluster add litmus-demo`