See: https://minikube.sigs.k8s.io/docs/handbook/network_policy/

```
minikube start
NS=fcs
kubectl create ns $NS
kubectl -n $NS apply -f manifest.yml

kubectl -n $NS exec -it web2 -- curl http://web1-service
kubectl -n $NS exec -it web3 -- curl http://web1-service

minikube stop
minikube delete

minikube start --cni=calico
watch kubectl get pods -l k8s-app=calico-node -A

kubectl create ns $NS
kubectl -n $NS apply -f manifest.yml

kubectl -n $NS exec -it web2 -- curl --connect-timeout 5 http://web1-service
kubectl -n $NS exec -it web3 -- curl --connect-timeout 5 http://web1-service
```
