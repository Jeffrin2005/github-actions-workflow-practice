# The Ultimate Kubectl Command Practice Guide

This document contains every single command we discussed, practiced, and ran during our entire Kubernetes masterclass. Use this as your final review before the interview!

---

## 1. Cluster & Connection Management
Commands to check if your cluster is alive and if you are connected to the right one.
*   `kubectl get nodes` : Checks if your cluster nodes (like `docker-desktop`) are ready.
*   `kubectl config current-context` : Tells you which cluster `kubectl` is currently talking to.
*   `kubectl config use-context docker-desktop` : Forces `kubectl` to talk to your local Docker cluster (fixes connection refused errors).

---

## 2. Resource Creation & Deletion (Declarative)
Commands to push your YAML files to the cluster or destroy them.
*   `kubectl apply -f secret.yml` : Sends a YAML file to the cluster to create or update resources.
*   `kubectl apply -f backend.yml` : (Remember to save the file first, and watch out for missing `---` separators!)
*   `kubectl delete -f frontend.yml` : Destroys everything created by that YAML file (the correct way to "stop" containers).

---

## 3. Gathering Information (The "Get" Commands)
Commands to see what is currently running in your cluster.
*   `kubectl get pods` : Lists all running pods in the current namespace.
*   `kubectl get svc` : Lists all services (ClusterIP, NodePort, LoadBalancer).
*   `kubectl get secrets` : Shows your created secrets.
*   `kubectl get deploy` : Lists all deployments.
*   `kubectl get all` : Lists every resource in the namespace at once.
*   `kubectl get pods -A` : Lists pods across *all* namespaces.
*   `kubectl get pods -o wide` : Shows extra details like the pod's IP and node.
*   `kubectl get deployment hr-backend -o yaml` : Outputs the live configuration of the deployment in YAML format.

---

## 4. Troubleshooting (The "Detective" Commands)
The absolute most important commands for a DevOps interview.
*   `kubectl logs <pod-name>` : Prints the standard output/console from *inside* the container. Used to check if the app connected to the database.
*   `kubectl describe pod <pod-name>` : Shows Kubernetes events *outside* the container. Used to debug `ErrImagePull`, `Pending`, or `CrashLoopBackOff` errors.
*   `kubectl describe deployment hr-backend` : Shows deployment details. Used to verify exactly which image version (`Image: jeffrinjojo/hr-backend:10`) is currently running.

---

## 5. Live Actions & Interactivity
Commands to modify live resources without editing YAML files.
*   `kubectl delete pod <pod-name>` : Manually kills a pod to test if a Deployment's self-healing will instantly recreate it.
*   `kubectl scale deployment hr-backend --replicas=5` : Instantly forces a deployment to scale up to 5 pods.
*   `kubectl scale deployment hr-backend --replicas=0` : Kills all pods, effectively "pausing" the deployment without deleting it.
*   `kubectl port-forward svc/frontend-service 8080:80` : Creates a temporary tunnel from your laptop's port 8080 to the cluster's port 80. Great for fast developer testing.
*   `kubectl exec -it <pod-name> -- /bin/sh` : Drops you into an interactive Linux terminal *inside* the running container.

---

## 6. Advanced Operations (Rollbacks)
Senior-level commands for fixing broken production environments with zero downtime.
*   `kubectl rollout history deployment/hr-backend` : Shows the history and revision numbers of a deployment.
*   `kubectl rollout undo deployment/hr-backend` : Instantly rolls back the deployment to the previous working version.

---

## 7. The Imperative "Cheat Codes" (Generating YAML)
How to write perfect YAML without actually typing it.
*   `kubectl run my-test-pod --image=nginx --dry-run=client -o yaml > pod.yml` : Generates a Pod YAML file without creating it in the cluster.
*   `kubectl create deployment my-test-deploy --image=nginx --dry-run=client -o yaml > deploy.yml` : Generates a Deployment YAML file.

---

## 8. Bonus Advanced Commands
*   `kubectl top pods` : Shows live CPU and RAM usage of pods (used when testing Horizontal Pod Autoscalers).
*   `kubectl edit deployment hr-backend` : Opens the live deployment YAML in a terminal editor for emergency, on-the-fly fixes.
