See:
* https://minikube.sigs.k8s.io/docs/handbook/network_policy/
* https://kubernetes.io/docs/concepts/services-networking/network-policies/
* https://projectcalico.docs.tigera.io/security/tutorials/kubernetes-policy-basic

```
RED='\033[1;31m'
GREEN='\033[1;32m'
NC='\033[0m' # No Color

check_ok() {
    if (( $? == 0 )); then
        echo -e "  ${GREEN}[PASS]${NC}"
    else
        echo -e "  ${RED}[FAIL]${NC}"
    fi
}

check_nok() {
    if (( $? != 0 )); then
        echo -e "  ${GREEN}[PASS]${NC}"
    else
        echo -e "  ${RED}[FAIL]${NC}"
    fi
}

run_tests() {
    echo "web2 --> web1 ; Should succeed:"
    kubectl -n $NS exec -it web2 -- curl --connect-timeout 5 http://web1 &>/dev/null
    check_ok

    echo "web3 --> web1 ; Should fail:"
    kubectl -n $NS exec -it web3 -- curl --connect-timeout 5 http://web1 &>/dev/null
    check_nok

    echo "web1 --> web2 ; Should fail:"
    kubectl -n $NS exec -it web1 -- curl --connect-timeout 5 http://web2 &>/dev/null
    check_nok

    echo "web1 --> web3 ; Should succeed:"
    kubectl -n $NS exec -it web1 -- curl --connect-timeout 5 http://web3 &>/dev/null
    check_ok

    echo "web3 --> web2 ; Should succeed:"
    kubectl -n $NS exec -it web3 -- curl --connect-timeout 5 http://web2 &>/dev/null
    check_ok
}


minikube start
export NS=fcs
kubectl create ns $NS
kubectl -n $NS apply -f manifest.yml

run_tests

minikube stop
minikube delete


minikube start --cni=calico
watch kubectl get pods -l k8s-app=calico-node -A

kubectl create ns $NS
kubectl -n $NS apply -f manifest.yml

run_tests

kubectl -n $NS delete -f manifest.yml
kubectl delete ns $NS
minikube stop
minikube delete
```
