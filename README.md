# Create an Ingress Controller:

## Create a namespace for your ingress resources
```
kubectl create namespace ingress-basic
```
## Add the ingress-nginx repository
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
## Use Helm to deploy an NGINX ingress controller
```
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux
```
## To get the public IP address, use the kubectl get service command.
```
kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller
```
##It takes a few minutes for the IP address to be assigned to the service.
```
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller
```
**output**
```
NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.
kubernetes.io/name=ingress-nginx

````

# Postgres_DB HA
```
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install -n keycloak keycloak-db bitnami/postgresql-ha
```
# keycloak-deployment

## Deployment YAML FILE:
```
kubectl apply -n ingress-basic -f keycloak-ingress.yaml
```
## Generate TLS certificates:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -out aks-ingress-tls.crt \
    -keyout aks-ingress-tls.key \
    -subj "/CN=demo.azure.com/O=aks-ingress-tls"
```
## Create Kubernetes Secret for it:

```  
kubectl create secret tls aks-ingress-tls \
    --namespace ingress-basic \
    --key aks-ingress-tls.key \
    --cert aks-ingress-tls.crt
```

## Create Ingress Resource
```
kubectl apply -f keycloak-ingress.yaml
```

## Test Connection
```
curl -v -k --resolve demo.azure.com:443:20.72.185.249 https://demo.azure.com/
```
