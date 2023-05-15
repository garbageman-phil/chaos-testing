1. install mini-kube - https://minikube.sigs.k8s.io/docs/start/  `minikube start`
2. add additional worker nodes
`minikube node add --worker`
3. setup load balancer - https://minikube.sigs.k8s.io/docs/start/
```commandline
kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080
```
`minikube tunnel`

4. Install deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-1
  name: nginx-1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-1
    spec:
      containers:
      - command:
        - /agnhost
        - netexec
        - --http-port
        - "8080"
        image: registry.k8s.io/e2e-test-images/agnhost:2.39
        name: my-pod
status: {}
```

5. install litmus 
`kubectl apply -f https://litmuschaos.github.io/litmus/3.0.0-beta6/litmus-3.0.0-beta6.yaml`
https://docs.litmuschaos.io/docs/getting-started/installation
6. check dns is working.  if not
```commandline
kubectl patch deployment coredns -n kube-system --patch '{"spec":{"template":{"spec":{"volumes":[{"name":"emptydir-tmp","emptyDir":{}}],"containers":[{"name":"coredns","volumeMounts":[{"name":"emptydir-tmp","mountPath":"/tmp"}]}]}}}}' 
```

7. change litmusportal-frontend-service from NodePort to LoadBalancer.  site should be accessible via 127.0.0.1:9091

8. deploy podtato
kubectl apply -f https://raw.githubusercontent.com/cncf/podtato-head/main/delivery/kubectl/manifest.yaml
https://github.com/podtato-head/podtato-head/tree/main/delivery/kubectl

  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  volumeMode: Filesystem