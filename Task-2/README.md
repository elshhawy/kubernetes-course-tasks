# Task 2 — FastAPI + MySQL on Kubernetes

## Overview

Deployed a FastAPI Guestbook application with MySQL on a three-node Kubernetes cluster.

## Environment

- Kubernetes v1.37.2
- kubeadm cluster: 1 control-plane and 2 workers
- containerd and Flannel
- FastAPI and MySQL
- NGINX Ingress

## Kubernetes Resources

- Namespace: webapp-stack
- ConfigMap: webapp-config
- Secret: mysql-secret
- PersistentVolume: mysql-pv (2Gi)
- PersistentVolumeClaim: mysql-pvc
- MySQL Deployment: 1 replica
- FastAPI Deployment: 2 replicas
- Services: mysql and webapp-api
- Ingress: webapp-ingress

## Architecture

Client -> NGINX Ingress -> webapp-api Service -> FastAPI Pods -> mysql Service -> MySQL Pod -> PVC -> PV

## Verification

- All three Kubernetes nodes reached Ready.
- MySQL Deployment: 1/1 Running.
- FastAPI Deployment: 2/2 Running.
- PV and PVC: Bound.
- Ingress successfully routes HTTP traffic.
- /health returned HTTP 200.
- /api/info returned application and database information.
- GET and POST /api/messages worked successfully.

## Persistence Test

Created a message through the API, deleted the MySQL Pod, waited for its replacement, and verified that the message was still available.

## Access

http://192.168.122.68:31513

## Screenshots

- `01-cluster-and-pods.png`: Cluster nodes and application Pods.
- `02-storage-pv-pvc.png`: PersistentVolume and PersistentVolumeClaim.
- `03-services-and-ingress.png`: Services and Ingress resources.
- `04-api-tests.png`: API health, info, and messages endpoints.

## Notes

The deployment uses a three-node kubeadm cluster, not Minikube. Database credentials are stored in a Kubernetes Secret and are not documented here.
