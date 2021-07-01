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

helm install -n ingress-basic keycloak-db bitnami/postgresql-ha
```
# keycloak-deployment

## Deployment YAML FILE:
```
kubectl apply -n ingress-basic -f keycloak.yaml
```
# There are 2 ways to create tls through ingress
# Option-1

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
**OR**
# Option-2
## Use Cert-Manager

## Step-1 (Install Cert-Manager)
```
** To install the cert-manager controller:**

Copy
# Label the ingress-basic namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace ingress-basic \
  --set installCRDs=true \
  --set nodeSelector."kubernetes\.io/os"=linux \
  --set webhook.nodeSelector."kubernetes\.io/os"=linux \
  --set cainjector.nodeSelector."kubernetes\.io/os"=linux

```
## Step-2 (Create a cluster Issuer)
```
kubectl apply -f cluster_issuer.yaml
```
## Step-3 Check certificate generated
```
kubectl get certificate --namespace ingress-basic
```
## Create Ingress Resource
```
kubectl apply -f keycloak-ingress.yaml
```

## Test Connection
```
curl -v -k --resolve demo.azure.com:443:EXTERNAL_IP https://demo.azure.com/
```
