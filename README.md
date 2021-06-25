# Inter Pod affinity and anti-affinity

## Create kind cluster

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
# One control plane node and three "workers".
#
# While these will not add more real compute capacity and
# have limited isolation, this can be useful for testing
# rolling updates etc.
#
# The API-server and other control plane components will be
# on the control-plane node.
#
# You probably don't need this unless you are testing Kubernetes itself.
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
- role: worker
```

## Set labels to node

```bash
kind create cluster --config kind/config.yaml --name affinity
kubectl get nodes --show-labels
kubectl label nodes affinity-worker safety=SWAL3
kubectl label nodes affinity-worker2 safety=SWAL3
kubectl label nodes affinity-worker3 safety=SWAL2
kubectl label nodes affinity-worker3 safety=SWAL2
kubectl label nodes affinity-worker2 session=mission-critical
kubectl label nodes affinity-worker3 session=mission-critical
kubectl get nodes --show-labels
```

## Testing applying pod to specific node according to label

```bash
kubectl apply -f kubernetes/nginx-swal2.yaml # Should deploy to affinity-worker3
kubectl get pods -o wide

kubectl apply -f kubernetes/nginx-swal3.yaml
# Should deploy to affinity-worker2 or affinity-worker
kubectl get pods -o wide
```

## Testing applying deployment to specific node according to deployment and node labels using anti affinity

```bash
kubectl apply -f kubernetes/nginx-with-pod-anti-affinity.yaml
# Should deploy to nodes with swal3 level and to
kubectl get nodes --show-labels
```

Result:

```
NAME                     STATUS   ROLES                  AGE   VERSION   LABELS
affinity-control-plane   Ready    control-plane,master   70m   v1.20.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=affinity-control-plane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=
affinity-worker          Ready    <none>                 69m   v1.20.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=affinity-worker,kubernetes.io/os=linux,safety=SWAL3
affinity-worker2         Ready    <none>                 69m   v1.20.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=affinity-worker2,kubernetes.io/os=linux,safety=SWAL3
affinity-worker3         Ready    <none>                 69m   v1.20.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=affinity-worker3,kubernetes.io/os=linux,safety=SWAL3
affinity-worker4         Ready    <none>                 69m   v1.20.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=affinity-worker4,kubernetes.io/os=linux,safety=SWAL3
```

```
kubectl get pods -o wide
```

Result:

```
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE               NOMINATED NODE   READINESS GATES
nginx-7dbbf579f-b4fnw   1/1     Running   0          14s   10.244.3.6   affinity-worker    <none>           <none>
nginx-7dbbf579f-gf2sx   1/1     Running   0          14s   10.244.1.5   affinity-worker2   <none>           <none>
```


## Testing applying deployment to specific node according to deployment and multiple node labels

```bash
kubectl apply -f kubernetes/nginx-with-pod-anti-affinity-and-node-affinity.yaml # Should deploy to affinity-worker3
kubectl get pods -o wide

# Should deploy to affinity-worker2 or affinity-worker
kubectl get pods -o wide
```


## Cleanup

```bash
kubectl delete pod/nginx-swal2
kubectl delete pod/nginx-swal3
kubectl delete deploy/nginx
```

