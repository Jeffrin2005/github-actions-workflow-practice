# Mastering `kubectl` Commands

To master Kubernetes commands for an interview, you need to understand the structure of the `kubectl` tool. Almost every command follows this pattern:
`kubectl [action] [resource] [resource_name]`

---

## 1. The "Get" Commands (Information Gathering)
These are your bread and butter. You use them to see what exists in your cluster.

*   `kubectl get pods` - Lists all pods in the current namespace.
*   `kubectl get svc` - Lists all services.
*   `kubectl get deploy` - Lists all deployments.
*   `kubectl get ingress` - Lists all ingress resources.
*   `kubectl get all` - Lists everything (pods, services, deployments, replicasets) in the namespace.

**Interview Tricks (Flags):**
*   `-n <namespace>` : By default, commands run in the `default` namespace. If an interviewer says "The pod is in the 'production' namespace," you MUST append `-n production`. (e.g., `kubectl get pods -n production`).
*   `-A` or `--all-namespaces` : Lists the resource across EVERY namespace. (e.g., `kubectl get pods -A`).
*   `-o wide` : Gives you extra details, like the internal IP address of the pod and which Node it is running on. (e.g., `kubectl get pods -o wide`).
*   `-o yaml` : Spits out the raw YAML code for a running resource. Very useful for backing up or checking exact configurations. (e.g., `kubectl get pod my-pod -o yaml`).

---

## 2. The "Troubleshooting" Commands (CRITICAL)
If things break, these are the commands you use to fix them.

*   `kubectl describe pod <pod-name>` : Shows incredibly detailed information about a pod, including its events (bottom of the output). **Use this when a pod won't start (Pending, ImagePullBackOff).**
*   `kubectl logs <pod-name>` : Prints the standard output/error from the application running inside the container. **Use this when a pod crashes (CrashLoopBackOff).**
*   `kubectl logs <pod-name> -c <container-name>` : Use this if a single pod has multiple containers and you need logs from a specific one.
*   `kubectl exec -it <pod-name> -- /bin/sh` : Drops you into an interactive terminal INSIDE the running container. Great for testing network connectivity (like running `curl` or `ping` from inside the pod).

---

## 3. The "Action" Commands (Creating, Deleting, Scaling)

*   `kubectl apply -f <filename.yml>` : The golden command. Creates or updates resources based on the YAML file.
*   `kubectl delete -f <filename.yml>` : Destroys everything defined in that YAML file.
*   `kubectl delete pod <pod-name>` : Kills a specific pod. *(Note: If the pod is managed by a Deployment, the Deployment will instantly recreate it!)*
*   `kubectl scale deployment <deployment-name> --replicas=5` : Manually overrides the YAML and scales the deployment to 5 pods instantly.

---

## 4. The "Imperative" Commands (Generating YAML)
Sometimes you don't want to write a YAML file from scratch. You can ask `kubectl` to generate the skeleton for you!

*   `kubectl run my-nginx --image=nginx --dry-run=client -o yaml > pod.yml` : Creates a perfect Pod YAML file without actually creating the pod in the cluster!
*   `kubectl create deployment my-deploy --image=nginx --dry-run=client -o yaml > deploy.yml` : Creates a Deployment YAML file.
