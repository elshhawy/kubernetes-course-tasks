# Task 1: Multi-container Pods (Init Container & Sidecar)

## Overview

This task demonstrates how to run multiple containers in a Kubernetes Pod using:

- Init Container for startup initialization.
- Nginx as the main application container.
- Sidecar Container for dynamic timestamp updates.
- Shared process namespace between containers.

## Environment

- Kubernetes: v1.37.0
- Minikube: v1.39.0
- Container Runtime: containerd
- CNI: Calico v3.32.2
- Cluster: 3 Nodes (1 Control Plane + 2 Workers)
- OS: Ubuntu 24.04 on WSL2

## Architecture

- Namespace: `webapp`
- Deployment: `web-deployment`
- Replicas: 3
- Pod Label: `app: web`

Each Pod contains:

| Container | Image | Role |
|---|---|---|
| init-done | busybox:1.36 | Runs before application containers |
| nginx | nginx:1.27 | Serves the web page on port 80 |
| time-updater | busybox:1.36 | Updates the timestamp every 5 seconds |


## Container Details

### 1. Init Container

The `init-done` container uses `busybox:1.36` and executes `echo done`.

It must finish successfully before the regular application containers start.

### 2. Main Container (Nginx)

The `nginx` container uses `nginx:1.27` and serves the HTML page on port 80.

### 3. Sidecar Container (Time Updater)

The `time-updater` container uses `busybox:1.36`.

It finds the Nginx master process using `pgrep -o nginx`, retrieves its PID, and updates the HTML file through `/proc/$PID/root/usr/share/nginx/html/index.html` every 5 seconds.

The Sidecar requires the `SYS_PTRACE` capability.


## Deployment

The application was deployed in the `webapp` namespace using the `web-deployment` Deployment with 3 replicas.

The manifest is available at `manifests/deployment.yaml`.

### Useful Commands

- Create namespace: `minikube kubectl -- create namespace webapp`
- Apply manifest: `minikube kubectl -- apply -f Task-1/manifests/deployment.yaml`
- Check Deployment: `minikube kubectl -- get deployment web-deployment -n webapp`
- Check Pods: `minikube kubectl -- get pods -n webapp -l app=web -o wide`

## Accessing the Web Application

The application was accessed through Kubernetes port forwarding:

`minikube kubectl -- port-forward -n webapp deployment/web-deployment 8080:80`

Open `http://localhost:8080` in a browser.

The page displays the MLOPS MENA COMMUNITY message with a dynamically updated timestamp.


## Failure Scenario: Init Container

The Init Container command was temporarily changed to:

`command: ["sh", "-c", "echo 'init failed'; exit 1"]`

After applying the modified manifest:

- The new Pod entered `Init:Error`.
- The Init Container restarted repeatedly.
- The new Pod eventually entered `Init:CrashLoopBackOff`.
- Nginx and the Sidecar did not start in the failed Pod.
- The three previously running Pods remained available.
- The website remained accessible during the failure.

This demonstrated the importance of Init Containers and the availability behavior of Kubernetes rolling updates.

## Recovery

The Init Container command was restored to:

`command: ["sh", "-c", "echo done"]`

The corrected manifest was validated and applied.

The Deployment rollout completed successfully.

Final Deployment status:

- READY: 3/3
- UP-TO-DATE: 3
- AVAILABLE: 3

Final Pod status:

- 3 Pods running.
- Each Pod has 2/2 regular containers ready.
- No restarts in the final observed state.
- Pods were distributed across the three cluster nodes.


## Screenshots

All screenshots are stored in the `screenshots/` directory.

| Screenshot | Description |
|---|---|
| [03-three-pods-running](screenshots/03-three-pods-running.png) | Three running Pods |
| [05-nginx-webpage](screenshots/05-nginx-webpage.png) | Nginx web page |
| [06-sidecar-timestamp-update](screenshots/06-sidecar-timestamp-update.png) | Dynamic timestamp update |
| [07-init-container-failure](screenshots/07-init-container-failure.png) | Init Container failure |
| [07-sidecar-processes](screenshots/07-sidecar-processes.png) | Shared process namespace |
| [08-init-container-failure-retries](screenshots/08-init-container-failure-retries.png) | Init Container retries |
| [09-init-container-crashloop](screenshots/09-init-container-crashloop.png) | CrashLoopBackOff state |
| [10-website-during-init-failure](screenshots/10-website-during-init-failure.png) | Website available during failure |
| [11-deployment-recovered](screenshots/11-deployment-recovered.png) | Successful Deployment recovery |
| [12-three-pods-final](screenshots/12-three-pods-final.png) | Final healthy Pods |

## Key Learnings

- Understanding Init Container execution order.
- Implementing the Sidecar pattern.
- Sharing a process namespace between containers.
- Inspecting processes using `ps` and `pgrep`.
- Understanding container restart behavior and CrashLoopBackOff.
- Observing Kubernetes Deployment rolling updates.
- Verifying application availability during a failed update.
- Recovering a Deployment by correcting its Pod specification.


## Pod Configuration Notes

The Pod enables a shared process namespace:

`shareProcessNamespace: true`

This allows the Sidecar to inspect the Nginx process and locate its master PID.

No application volumes are defined or mounted in this task. The Sidecar accesses the Nginx document root through the shared process namespace and `/proc`.

