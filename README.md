1. install mini-kube - https://minikube.sigs.k8s.io/docs/start/  `minikube start`
2. add additional worker nodes with worker label
`minikube node add --worker`
`node-role.kubernetes.io/worker: ""`
3. setup load balancer - https://minikube.sigs.k8s.io/docs/start/
```commandline
kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080
```
`minikube tunnel`

4. check dns is working.  if not
```commandline
kubectl patch deployment coredns -n kube-system --patch '{"spec":{"template":{"spec":{"volumes":[{"name":"emptydir-tmp","emptyDir":{}}],"containers":[{"name":"coredns","volumeMounts":[{"name":"emptydir-tmp","mountPath":"/tmp"}]}]}}}}' 
```

5. make ns
`k create ns nginx-1`
`k create ns nginx-2`

6. Install deployment
k create -f ./nginx-1/deployment.yaml
k create -f ./nginx-2/deployment.yaml


7. install litmus 
`kubectl apply -f https://litmuschaos.github.io/litmus/3.0.0-beta6/litmus-3.0.0-beta6.yaml`
https://docs.litmuschaos.io/docs/getting-started/installation


7. change litmusportal-frontend-service from NodePort to LoadBalancer.  site should be accessible via 127.0.0.1:9091

### create app / service chaos
1. annotate deployment to enable chaos
`k annotate deploy/nginx-1 litmuschaos.io/chaos="true" -n nginx-1`

1. install pod-delete experiment
k apply -f https://hub.litmuschaos.io/api/chaos/3.0.0-beta6?file=charts/generic/pod-delete/experiment.yaml -n nginx-1

1. add litmus sa in app ns
k create -f ./nginx-1/pod-delete-sa.yaml

1. create engine in app ns



### create platform specific chaos
Setup, litmus should have its own dedicated nodes as the it could "knock the stool from under itself".  not sure how well this works w static pods.

1. [node drain](https://hub.litmuschaos.io/generic/node-drain)
experiment: `kubectl apply -f https://hub.litmuschaos.io/api/chaos/3.0.0-beta6?file=charts/generic/node-drain/experiment.yaml -n litmus`
rbac: `k apply -f ./experiments/node-drain/rbac-node-drain.yaml`

types of probes / validations to prove successul might be:
- check for pods restarting
- check for same number of deployments running by label
- check deployments for correct number of pods ie running pods = replicas
- healthcheck or http probe success

