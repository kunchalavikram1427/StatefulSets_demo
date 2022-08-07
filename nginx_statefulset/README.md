# StatefulSet Example
https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pods

## Pod DNS(A/AAAA records)
In general a pod has the following DNS resolution:
pod-ip-address.my-namespace.pod.cluster-domain.example.
For example, if a pod in the default namespace has the IP address 172.17.0.3, and the domain name for your cluster is cluster.local, then the Pod has a DNS name:
172-17-0-3.default.pod.cluster.local.
Any pods exposed by a Service have the following DNS resolution available:
pod-ip-address.service-name.my-namespace.svc.cluster-domain.example.

Pods within a statefulset controller, gets DNS resolution
pod-name.headless-service-name.my-namespace.svc.cluster.local

## Demo Example
This demo uses a simple nginx webserver

```
kubectl apply -f web.yaml
kubectl get pods -l app=nginx
kubectl get service nginx-headless nginx-normal
kubectl get statefulset web

for i in 0 1; do kubectl exec "web-$i" -c nginx -- sh -c 'hostname'; done
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
nslookup nginx-normal
nslookup nginx-headless
nslookup web-0.nginx-headless

```
