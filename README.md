# Alliance Auth in Kubernetes

## Pods
- Redis
- Alliance Auth
  - Alliance Auth App (Python etc.)
  - Nginx (static assets)

## Assumptions / Notes
- You already have ingress setup in your cluster (I'm using Traefik 2.1, modify/create ingress as needed to expose app to users)
- This was tested on Digital Ocean's managed kubernetes. You may need to tweak the volume claims on a different provider.
- Based on using a tagged image of AA, currently `mckernanin/alliance-auth-2.6.5`
- I have migrations set to run on container start, so that when you update the tag you'll run any migrations automatically

## Getting Started
1. Create namespace for project: `kubectl create namespace alliance-auth-k8s`
1. Fill out env vars in `deployment.yaml`
1. Deploy all files: `kubectl apply -f app`

## Commands
You will need to specify the current running pod in all commands, replacing `%%POD%%` with the current pod name
- AA Logs: `kubectl logs -f %%POD%% alliance-auth`
- NGINX Logs: `kubectl logs -f %%POD%% nginx`
- Exec (SSH) into running container: `kubectl exec -it %%POD%% -- bash`

## Installing packages / etc
- You can add commands to the command section of the AA container config to install packages, disable running migrations on start, etc.
- You can install packages in the running container by exec, but keep in mind that they will not persist if the container is restarted.
- To deploy a new config, edit the configmap in `deployment.yml` and then restart the container. Just applying a new config will not restart things, I use a script like this:
```bash
#!/bin/bash
kubectl scale deployment alliance-auth --replicas=0 && kubectl apply -f app/deployment.yml
kubectl scale deployment alliance-auth --replicas=1
```
