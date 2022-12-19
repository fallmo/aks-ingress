# Ingress Nginx On Azure
Documentation URL: https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli

## 1. Install ingress-nginx with `helm` using azure-cli
```bash
NAMESPACE=ingress-basic
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace $NAMESPACE \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz
```

## 2. Wait for complete installation
> Ensure all pods are STATUS=RUNNING
```bash
kubectl get pods -n $NAMESPACE -w
```
> Ensure the Load Balancer <i>serivce</i> has been given an External IP Address
```
kubectl get services --namespace $NAMESPACE -o wide
```

## 3. Test Installation
Deploy the test application w/ ingress using yaml below
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-one
  namespace: test 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-one
  template:
    metadata:
      labels:
        app: aks-helloworld-one
    spec:
      containers:
      - name: aks-helloworld-one
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "AKS Ingress Demo"
---
apiVersion: v1
kind: Service
namespace: test
metadata:
  name: aks-helloworld-one  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-one
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: test
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: aks-helloworld-one
            port:
              number: 80
```
