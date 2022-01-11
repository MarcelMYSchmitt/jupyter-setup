# Introduction

Helm Chart and docker-compose setup for running JupyterHub in Kubernetes (on cloud / on premise) or locally. We are using NGINX and JupyterHub non root images. NGINX is used as reverse proxy so we can run JupyterHub by using a relative path `/jupyter`. 

<br /> 

## Setup Docker

Run the containers locally by using `docker-compose up`. Before running you have to create a docker network, otherwise you will get following error message. 

```
ERROR: Network jupyter-network declared as external, but could not be found. Please create the network manually using `docker network create jupyter-network` and try again.
```

The network is needed because NGINX is going to route to JupyterHub by using the docker service name (service name discovery). You can then access JupyterHub via `localhost:8080/jupyter`. 

<br /> 

## Setup Kubernetes

You can use the Helm Charts. You have to adjust the values inside `values.yaml` like setting the correct secrets for the pull secret or the token. 

For accessing via URL you have to create a separate ingress (NGINX, Traefik and so on) or Virtual Service (Istio). 

Here is an example for a virtual service yaml file. 
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: application
  namespace: istio-system
spec:
  gateways:
    - istio-system/gateway
  hosts:
    - "baseurl.com"
  exportTo:
    - "."
  http:
    - match:
        - uri:
            prefix: /jupyter
      route:
        - destination:
            host: jupyter-notebook.ns-application-environment.svc.cluster.local
            port:
              number: 80
          weight: 100
```

<br /> 


## Non root containers

You can find the non root containers on Docker Hub. With non root containers you have the same setup locally and on clusters where non root containers are not allowed.

JUPYTERHUB
```
https://hub.docker.com/r/jupyter/minimal-notebook  
jupyter/minimal-notebook:latest
```


NGINX 
```
https://hub.docker.com/r/nginxinc/nginx-unprivileged
nginxinc/nginx-unprivileged:1.17.5-alpine
```