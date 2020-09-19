# keycloak-k8s
Keycloak SSO on DigitalOcean kubernetes

## Install ingress-nginx
Using https://kubernetes.github.io/ingress-nginx/deploy/

### install command for DigitalOcean:
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.35.0/deploy/static/provider/do/deploy.yaml`

(you can also use helm to install... see bottom of above linked docs)

### check install
`kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch`

### check ingress version
```
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
```
Output:
```
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v0.35.0
  Build:         54ad65e32bcab32791ab18531a838d1c0f0811ef
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.2

-------------------------------------------------------------------------------
```

## Install cert-manager
`arkade install cert-manager`

## Create namespace for keycloak
`kubectl create namespace keycloak`

## Install certificate issuer for cert-manager
```
# letsencrypt-issuer.yaml
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: letsencrypt-staging
  namespace: keycloak
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: <your email>m
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource used to store the account's private key.
      name: keycloak-staging-issuer-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
---
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: letsencrypt-prod
  namespace: keycloak
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: <your email>m
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource used to store the account's private key.
      name: keycloak-prod-issuer-key
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
    - http01:
        ingress:
          class: nginx
```

`kubectl apply -f letsencrypt-issuer.yaml`

## Edit keycloak-config.yaml
- Create a username/password for the keycloak admin user
- Use your own domain
- Specify the letsencrypt-staging issuer for testing
- When it works, uninstall keycloak, change to letsencrypt-prod issuer and reinstall

```
# keycloak-config.yaml
extraEnv: |
  - name: KEYCLOAK_USER
    value: <put username here>
  - name: KEYCLOAK_PASSWORD
    value: <put password here>
postgresql:
  enabled: false
ingress:
  enabled: true
  rules:
    - host: keycloak.example.com
      paths:
      - /
  tls:
    - hosts:
      - keycloak.example.com
      secretName: keycloak-tls
  annotations:
    cert-manager.io/issuer: letsencrypt-staging
```

## Install Keycloak v11 using CodeCentric helm chart
```
helm repo add codecentric https://codecentric.github.io/helm-charts
helm install keycloak codecentric/keycloak \
  --atomic \
  --version 9.1.0 \
  --namespace keycloak \
  --values keycloak-config.yaml
```

# Uninstall keycloak
`helm uninstall keycloak --namespace keycloak`


