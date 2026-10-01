# Kubernetes Interview Prep

## 1. Kubernetes Fundamentals & YAML

Kubernetes (K8s) is a container orchestration platform that automates deployment, scaling, and management of containerized applications.

### Core Components
*   **Control Plane (Master Node):**
    *   **API Server:** The front end of the control plane. All communication goes through here.
    *   **etcd:** Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.
    *   **Scheduler:** Watches for newly created Pods with no assigned node, and selects a node for them to run on.
    *   **Controller Manager:** Runs controller processes (Node controller, Job controller, Endpoints controller, Service Account & Token controllers).
*   **Worker Nodes:**
    *   **Kubelet:** An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.
    *   **Kube-proxy:** Maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
    *   **Container Runtime:** The software that is responsible for running containers (e.g., containerd, CRI-O).

### Basic Objects
*   **Pod:** The smallest deployable units of computing that you can create and manage in Kubernetes. Usually encapsulates one container (sometimes tightly coupled multiple containers).
*   **Deployment:** Provides declarative updates for Pods and ReplicaSets. You describe a desired state, and the Deployment Controller changes the actual state to the desired state at a controlled rate.
*   **ReplicaSet:** Maintains a stable set of replica Pods running at any given time. Usually managed by a Deployment.
*   **Service:** An abstract way to expose an application running on a set of Pods as a network service.
*   **Namespace:** Provides a mechanism for isolating groups of resources within a single cluster.

### YAML Basics
Kubernetes resources are defined using YAML. Key fields in almost every K8s YAML:
*   `apiVersion`: Which version of the Kubernetes API you're using.
*   `kind`: What kind of object you want to create (e.g., Pod, Deployment, Service).
*   `metadata`: Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`.
*   `spec`: The desired state for the object.

**Example Pod YAML:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

---

## 2. Services

Services decouple the frontend and backend. Pods are ephemeral (they die and are recreated with new IP addresses). Services provide a stable IP address and DNS name for a set of Pods.

### Types of Services
1.  **ClusterIP (Default):** Exposes the Service on a cluster-internal IP. Makes the Service only reachable from within the cluster.
2.  **NodePort:** Exposes the Service on each Node's IP at a static port (the NodePort). You can contact the NodePort Service, from outside the cluster, by requesting `<NodeIP>:<NodePort>`.
3.  **LoadBalancer:** Exposes the Service externally using a cloud provider's load balancer.
4.  **ExternalName:** Maps the Service to the contents of the `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record.

**Example Service (ClusterIP) YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-service
spec:
  selector:
    app: nginx # Routes traffic to Pods with this label
  ports:
    - protocol: TCP
      port: 80 # Port exposed by the Service
      targetPort: 80 # Port the container is listening on
```

---

## 3. ConfigMaps and Secrets

These decouple configuration artifacts from image content to keep containerized applications portable.

### ConfigMaps
Used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

**Example ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "db.example.com"
  log_level: "INFO"
```

### Secrets
Used to store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. Storing confidential information in a Secret is safer and more flexible than putting it verbatim in a Pod definition or in a container image. (Values must be base64 encoded in the YAML, though they are stored securely by etcd).

**Example Secret:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4= # "admin" base64 encoded
  password: cGFzc3dvcmQxMjM= # "password123" base64 encoded
```

### Using them in a Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-app-container
    image: my-app:v1
    env:
      - name: DB_URL
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: database_url
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: db-credentials
            key: password
```

---

## 4. Kubernetes Troubleshooting

Troubleshooting is a critical skill for interviews. Follow a systematic approach.

### 1. Pod is Pending
*   **Cause:** Scheduler cannot find a node to place the pod.
*   **Action:**
    *   `kubectl describe pod <pod-name>` (Look at the Events at the bottom).
    *   Common issues: Not enough resources (CPU/Memory) on any node, NodeSelector/Affinity not matching any nodes, Taints on nodes that the pod doesn't tolerate.

### 2. Pod is CrashLoopBackOff
*   **Cause:** The container starts, crashes, and Kubernetes keeps trying to restart it.
*   **Action:**
    *   `kubectl logs <pod-name>` (Check application logs for errors).
    *   `kubectl logs <pod-name> --previous` (If it crashed, look at the previous instance).
    *   `kubectl describe pod <pod-name>` (Check for exit codes like OOMKilled - exit code 137, or application errors).
    *   Common issues: Application error (missing dependencies, bad config), OOMKilled (Out of Memory - increase memory limits), Liveness probe failing.

### 3. Pod is ImagePullBackOff / ErrImagePull
*   **Cause:** Kubelet cannot pull the container image.
*   **Action:**
    *   `kubectl describe pod <pod-name>`
    *   Common issues: Typo in image name or tag, Image doesn't exist in registry, Lack of authentication to a private registry (need an `imagePullSecret`).

### 4. Service not routing traffic
*   **Cause:** Misconfiguration between Service and Pods.
*   **Action:**
    *   Check Service labels: `kubectl get service <svc-name> -o yaml` (Look at `spec.selector`).
    *   Check Pod labels: `kubectl get pods --show-labels`. Ensure they match the Service selector.
    *   Check Endpoints: `kubectl get endpoints <svc-name>`. If this is empty, the Service isn't finding any Pods.
    *   Check targetPort: Ensure the Service `targetPort` matches the container's exposed port.

### Essential Troubleshooting Commands
*   `kubectl get pods -n <namespace>`
*   `kubectl describe pod <pod-name>`
*   `kubectl logs <pod-name>`
*   `kubectl exec -it <pod-name> -- /bin/sh` (To get inside the container and debug)
*   `kubectl get events --sort-by='.metadata.creationTimestamp'` (Cluster-wide events)
*   `kubectl get svc`
*   `kubectl get endpoints`
