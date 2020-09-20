# keycloak-k8s
Keycloak SSO on DigitalOcean kubernetes

## Prerequisites
- A running k8s cluster on DigitalOcean
- arkade installed

## Install ingress-nginx to the k8s cluster
Using instructions: https://www.openfaas.com/blog/openfaas-oidc-okta/
`arkade install ingress-nginx`

### Check install
`kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --watch`

## Install cert-manager
`arkade install cert-manager`

## Create namespace for keycloak
`kubectl create namespace keycloak`

## Create Keycloak deployment
`kubectl apply -f keycloak.yaml`

Reference: https://www.keycloak.org/getting-started/getting-started-kube

## Create certificate issuer for cert-manager
- Edit keycloak-issuer.yaml, changing:
  - "email" to your own email address

`kubectl apply -f keycloak-issuer.yaml`

## Create ingress for keycloak
- Edit keycloak-ingress.yaml, changing:
  - "hosts" (2 occurences) to your own domain 

`kubectl apply -f keycloak-ingress.yaml`