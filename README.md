# k8s-nginx-ingress-certmanager
Example for setting up k8s nginx ingress with cert manager and automatic LetsEncrypt certificates renewal, serve simple helloworld container page as an example.



# Deploy nginx-ingress chart

```
# I'm using Helmv2 here, consider Helmv3 in the future where tiller dependency is not needed:

kubectl apply -f https://raw.githubusercontent.com/mrx88/aks-deploy-example/master/kube/helm-rbac.yaml
helm init --service-account=tiller
helm install --namespace kube-system --name nginx-ingress stable/nginx-ingress --set rbac.create=true

or: 
helm install stable/nginx-ingress --namespace kube-system --set controller.replicaCount=2 --set service.annotations[0]="service.beta.kubernetes.io/azure-dns-label-name: mycustomdnsname"

# In case if IP whitelisting is needed, deploy ingress with  X-Forward-IP header support, increase controller pod count as well:
helm upgrade nginx-ingress stable/nginx-ingress --set controller.replicaCount=2 --set controller.service.externalTrafficPolicy=Local

# To use IP whitelisting, use the following annotation in k8s manifest:
 ingress.kubernetes.io/whitelist-source-range: <ip>/32
```


# K8s Namespaces

For testing, lets use different namespaces:
```
 kubectl create namespace dev
 kubectl create namespace prod
```
# Deploy cert-manager chart

```
# Install CRDs
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml


# Separate namespace
kubectl create namespace cert-manager
 
# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io
helm repo update
# Install the cert-manager helm chart
helm install --name cert-manager --namespace cert-manager --version v0.12.0 jetstack/cert-manager
```

# LetsEncrypt certificates

```
# Deploy Issuer (namespace specific)
kubectl create -f letsencrypt-staging.yaml
# Deploy Issuer (per cluster specific, done in this example)
kubectl apply -f cluster-issuer.yaml
kubectl get clusterissuer
```

# Deploy manifests
```
# dev namespace
kubectl apply -f deploy-test.yaml
# prod namespace
kubectl apply -f deploy-prod.yaml
```

# Verify

```
kubectl logs po/cert-manager-6464494858-pwzbc -n cert-manager
kubectl describe orders --all-namespaces
kubectl get certificates
kubectl describe certificates/test-staging-letsencrypt
kubectl logs nginx-ingress-controller-7d9f786f5-wp5bv -n kube-system
```

 # Documentations
 
https://docs.microsoft.com/en-us/azure/aks/ingress-tls

https://github.com/jetstack/cert-manager/tree/master/deploy/charts/cert-manager
```