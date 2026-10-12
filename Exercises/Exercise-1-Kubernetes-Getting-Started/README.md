# Exercise 1: First Kubernetes Pod with Minikube

## Scenario: A Small Storefront Service

As a **DevOps Engineer at Zepto**, the product team built a lightweight web app that shows the storefront and delivery status page for customers. The task is to deploy this app on Kubernetes so that it is always running, portable, and can be scaled later.

We simulate this using the `nginx` container image (representing Zepto's storefront web app).

## What I Practiced
I launched a first application inside Kubernetes and exposed it for browser access.

## Requirements
- Minikube installed
- kubectl installed
- Docker or VM driver (VirtualBox, Hyper-V) installed

## Walkthrough

### Step 1: Start Minikube
```bash
minikube start
```

### Step 2: Launch an Nginx Pod
```bash
kubectl run hello-k8s --image=nginx --port=80
```

### Step 3: Confirm the Pod is Ready
```bash
kubectl get pods
```

**Output:**
```
NAME         READY   STATUS    RESTARTS   AGE
hello-k8s    1/1     Running   0          10s
```

### Step 4: Create a Service for the Pod
```bash
kubectl expose pod hello-k8s --type=NodePort --port=80
```

### Step 5: Open the Service
```bash
minikube service hello-k8s
```

**Result:** The Nginx welcome page is displayed successfully.

## What Kubernetes Does Behind the Scenes

When we run `kubectl run hello-k8s --image=nginx --port=80`, the following happens internally:

1. **kubectl** sends the request to the **Kubernetes API Server**
2. The **API Server** stores the Pod specification in **etcd**
3. The **Scheduler** assigns the Pod to a node
4. The **kubelet** on the assigned node pulls the nginx image and starts the container
5. When we expose it as a NodePort service, **kube-proxy** sets up networking rules to route traffic

## Notes from the Exercise
- A **Pod** is the smallest deployable unit in Kubernetes
- `kubectl run` creates a Pod with the specified container image
- `kubectl expose` creates a Service to make the Pod accessible
- **NodePort** service type exposes the service on a static port on each node
- **Minikube** provides a local single-node Kubernetes cluster for development
