
### Option 1 
- use the ingress controller
`External ---> Ingress NginX ---> Cluster IP Service ---> Pod`


### Option 2
Use NodePort Service
`External ---> NodePort Service ---> Pod`

### Option 3
Set up manual port-forwarding for a specific Pod with kubectl (for Dev only)

```Powershell
kubectl get pods
###nats-depl-5cd8fc68c8-w4mt5

kubectl port-forward nats-depl-76c5576647-6ph7d 4222:4222
## First port is on local, second is port on the pod
```