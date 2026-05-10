kubectl exec -n rbac-demo -it <pod-name> -- sh

kubectl get pods -n rbac-demo ✅
kubectl get pods -n default   ❌
kubectl get nodes             ✅
kubectl run test --image=nginx❌

kubectl auth can-i --as=system:serviceaccount:rbac-demo:demo-sa
