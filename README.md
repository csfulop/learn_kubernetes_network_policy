See:
* https://minikube.sigs.k8s.io/docs/handbook/network_policy/
* https://kubernetes.io/docs/concepts/services-networking/network-policies/
* https://projectcalico.docs.tigera.io/security/tutorials/kubernetes-policy-basic

```
minikube start
NS=fcs
kubectl create ns $NS
kubectl -n $NS apply -f manifest.yml

check_ok() {
    if (( $? == 0 )); then
        echo "  [PASS]"
    else
        echo "  [FAIL]"
    fi
}

check_nok() {
    if (( $? != 0 )); then
        echo "  [PASS]"
    else
        echo "  [FAIL]"
    fi
}

echo "Should succeed:"
kubectl -n $NS exec -it web2 -- curl --connect-timeout 5 http://web1
check_ok

echo "Should fail:"
kubectl -n $NS exec -it web3 -- curl --connect-timeout 5 http://web1
check_nok

echo "Should fail:"
kubectl -n $NS exec -it web1 -- curl --connect-timeout 5 http://web2
check_nok

echo "Should succeed:"
kubectl -n $NS exec -it web1 -- curl --connect-timeout 5 http://web3
check_ok



minikube stop
minikube delete


minikube start --cni=calico
watch kubectl get pods -l k8s-app=calico-node -A

kubectl create ns $NS
kubectl -n $NS apply -f manifest.yml

echo "Should succeed:"
kubectl -n $NS exec -it web2 -- curl --connect-timeout 5 http://web1
check_ok

echo "Should fail:"
kubectl -n $NS exec -it web3 -- curl --connect-timeout 5 http://web1
check_nok

echo "Should fail:"
kubectl -n $NS exec -it web1 -- curl --connect-timeout 5 http://web2
check_nok

echo "Should succeed:"
kubectl -n $NS exec -it web1 -- curl --connect-timeout 5 http://web3
check_ok
```
