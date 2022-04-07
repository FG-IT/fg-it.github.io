---
title: Install Nginx Ingress Controller & Cert Manager on GKE
date: 2022-03-24 14:49:00
author: YZ
categories:
- kubenetes
tags:
- nginx ingress controller
- kubenetes
- GKE
- cert-manager
---

As more and more browsers turn off third-party cookies by default, it's harder and harder for third-party to track activities. Recently, I received a request to integrate [CJ affliate](https://www.cj.com/) to our e-commerce website. They requested to implement reverse proxy to turn the third-party cookies to first-party cookie. It seems there is no good way to do it using GKE's ingress controller since our e-commerce website is deployed in google cloud as a kubernetes cluster. After a few googling researches, I decided to install and use [nginx ingress controller](https://kubernetes.github.io/ingress-nginx/).

This post is somewhat written for a speciallized scenario. For a general guidance, please refer [DigitalOcean's tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes#step-4-installing-and-configuring-cert-manager). 

# Conventions
1. `fancyshop.com` is your website domain, and `www.fancyshop.com` is set to redirect to `fancyshop.com`
2. you have already created a service called `fancyshop` to serve all http requests.
3. if you are using autopilot cluster, make sure the version `>=1.21`, `cert-manager` doesn't work with GKE Autopilot mode version `<1.21`

# Installing NGINX Ingress Controller
basically, you just need to follow the [installation guide](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

for me, I chose to apply a YAML manifest.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/cloud/deploy.yaml
```

After sucessful apply the manifest, you should find a srevice called `	ingress-nginx-controller` in `Kubernetes Engine -> Service & Ingress` Service tab. The type is `External load balancer`, and there is an ip attached it, we name is `ingress-nginx-controller-ip`. 

# DNS
Please also reserve `ingress-nginx-controller-ip` ip address, and make changes to your dns configuration to point `fancyshop.com` to `ingress-nginx-controller-ip`

*note: if fancyshop.com is currently in production, please test it using another subdomain, something like tmp.fancyshop.com. After everything is done, finally point fancyshop.com to `ingress-nginx-controller-ip`*

# Creating Ingress Resource
Let's name the ingress file `fancyshop-ingress.yml` and fill it with the following yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx # this line is important, it makes sure the nginx ingress controller installed can find and apply this ingress resource.
    nginx.ingress.kubernetes.io/from-to-www-redirect: "true" # redirect www to non-www or redirect non-www to www.
    nginx.ingress.kubernetes.io/server-snippet: | # it's cj affliate's configuration for nginx. You don't have to add it.
      server_name fancyshop.com;
      location /proxydirectory/ {  
        proxy_ssl_server_name on;
        proxy_pass https://www.mczbf.com/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $server_name;
        proxy_set_header X-Forwarded-Request-Host $host;
        proxy_set_header X-Forwarded-Request-Path $request_uri;
      }
  name: fancyshop-nginx
  namespace: default
spec:
  rules:
  - host: fancyshop.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: fancyshop
            port:
              number: 80
```

apply it by running 
```bash
kubectl apply -f fancyshop-ingress.yml
```

You'll see the confirmation of the ingress's creation.

now, type `fancyshop.com` in your browser or makes a `curl` call, you are able to see your website.

# Installing and Configuring Cert-Manager
## Installing
The simplies way to install it is by applying a yaml manifest. However, the manifest has some issues with GKE, it seems the reason is a wrong namespace setting. 
I suggest you download the manifest to local from `https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.yaml`. And replaces all `kube-system` with `cert-manager`. There are total 6 replacements. and run `kubectl apply -f cert-manager.yaml`

To verify our installation, check the cert-manager Namespace for running pods:
```bash
kubectl get pods --namespace cert-manager
```
```
Output
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-578cd6d964-hr5v2              1/1     Running   0          99s
cert-manager-cainjector-5ffff9dd7c-f46gf   1/1     Running   0          100s
cert-manager-webhook-556b9d7dfd-wd5l6      1/1     Running   0          99s
```

## Creating Certificates Issuer
Open a file called `prod_issuer.yaml` in your favorite editor, and fill in the following manifest:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: [your_email_address_here]
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```

Roll out this Issuer using kubectl:
```bash
kubectl create -f prod_issuer.yaml
```
You should see the following output:
```
Output
clusterissuer.cert-manager.io/letsencrypt-prod created
```

After this step is done, there is no certificate created for any domain. We have to make some changes to our ingress resource.

# Issuing Production Let's Encrypt Certificates
make changes to our ingress manifest file as following:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod" # using letsencrypt-prod to issue certificates.
    kubernetes.io/ingress.class: nginx 
     nginx.ingress.kubernetes.io/from-to-www-redirect: "true"
    nginx.ingress.kubernetes.io/server-snippet: |
      server_name fancyshop.com;
      location /proxydirectory/ {  
        proxy_ssl_server_name on;
        proxy_pass https://www.mczbf.com/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $server_name;
        proxy_set_header X-Forwarded-Request-Host $host;
        proxy_set_header X-Forwarded-Request-Path $request_uri;
      }
  name: fancyshop-nginx
  namespace: default
spec:
  tls: # issue certificates for both fancyshop.com and www.fancyshop.com, and store the certificates in Secret fancyshop-prod-tls
  - hosts:
    - fancyshop.com
    - www.fancyshop.com
    secretName: fancyshop-prod-tls
  rules:
  - host: fancyshop.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: fancyshop
            port:
              number: 80
```

# Appendix
* [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
* [cert-manager](https://cert-manager.io/docs/installation/)
* [DigitalOcean Set Up an Nginx Ingress with Cert-Manager](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes#step-4-installing-and-configuring-cert-manager)