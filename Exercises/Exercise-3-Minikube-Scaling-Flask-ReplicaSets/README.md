# Exercise 3: Scaling and Recovering Flask Pods with a ReplicaSet

## Scenario: Handling a Flash-Sale Spike

During a flash sale (like Flipkart's Big Billion Days), traffic can spike from 100 to 10,000 requests/minute. A single Pod crashes under this load. Using **ReplicaSets**, Kubernetes can scale out to multiple Pods distributing the load, and scale back down after the sale.

## Lab Goals
- Understand ReplicaSets and Pods
- Scale Flask App deployment
- Observe pod distribution and self-healing

## Components Used
| File | Description |
|------|-------------|
| `app.py` | Flash Sale Flask app with `/`, `/buy`, `/health` endpoints |
| `Dockerfile` | Docker image using python:3.11-slim + gunicorn |
| `flashsale-replicaset.yaml` | ReplicaSet (3 replicas) + NodePort Service |

## Experiment Log

### Step 1: Reset the Local Cluster
```bash
minikube stop
minikube delete
minikube start --nodes=1
```

**Output:**
```
minikube v1.38.1 on Windows
Starting "minikube" primary control-plane node in "minikube" cluster
Done! kubectl is now configured to use "minikube" cluster
```

### Step 2: Confirm the Node
```bash
kubectl get nodes
```
**Output:**
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   42s   v1.35.1
```

### Step 3: Build the Flash-Sale Image in Minikube
```bash
minikube docker-env
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker build -t flashsale:1.0 .
```

**Output:**
```
Successfully built <image-id>
Successfully tagged flashsale:1.0
```

### Step 4: Create the ReplicaSet and Service
```bash
kubectl apply -f flashsale-replicaset.yaml
```
**Output:**
```
replicaset.apps/flashsale-rs created
service/flashsale-svc created
```

### Step 5: Check the Initial Replica Count
```bash
kubectl get rs
```
**Output:**
```
NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   3         3         3       30s
```

### Step 6: Confirm the Three Pods
```bash
kubectl get pods
```
**Output:**
```
NAME                 READY   STATUS    RESTARTS   AGE
flashsale-rs-8gbfp   1/1     Running   0          35s
flashsale-rs-f4gsl   1/1     Running   0          35s
flashsale-rs-nb5kl   1/1     Running   0          35s
```

### Step 7: Increase Capacity to Five Replicas
```bash
kubectl scale rs flashsale-rs --replicas=5
```
**Output:**
```
replicaset.apps/flashsale-rs scaled
```

### Step 8: Confirm the New Desired State
```bash
kubectl get rs
```
**Output:**
```
NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   5         5         5       2m
```

### Step 9: Confirm Five Pods Are Running
```bash
kubectl get pods
```
**Output:**
```
NAME                 READY   STATUS    RESTARTS   AGE
flashsale-rs-8gbfp   1/1     Running   0          2m
flashsale-rs-f4gsl   1/1     Running   0          2m
flashsale-rs-nb5kl   1/1     Running   0          2m
flashsale-rs-xk2pt   1/1     Running   0          30s
flashsale-rs-zt9mn   1/1     Running   0          30s
```

### Step 10: Remove One Pod to Test Recovery
```bash
kubectl delete pod flashsale-rs-f4gsl
```
**Output:**
```
pod "flashsale-rs-f4gsl" deleted
```

### Step 11: Watch the ReplicaSet Replace It
```bash
kubectl get pods
```
**Output:**
```
NAME                 READY   STATUS    RESTARTS   AGE
flashsale-rs-8gbfp   1/1     Running   0          3m
flashsale-rs-nb5kl   1/1     Running   0          3m
flashsale-rs-xk2pt   1/1     Running   0          1m
flashsale-rs-zt9mn   1/1     Running   0          1m
flashsale-rs-hqtm7   1/1     Running   0          5s   ← NEW pod auto-created
```

### Step 12: Inspect Pod Placement
```bash
kubectl get pods -o wide
```
**Output:**
```
NAME                 READY   STATUS    RESTARTS   AGE   IP            NODE
flashsale-rs-8gbfp   1/1     Running   0          4m    10.244.0.7    minikube
flashsale-rs-hqtm7   1/1     Running   0          1m    10.244.0.12   minikube
flashsale-rs-nb5kl   1/1     Running   0          4m    10.244.0.9    minikube
flashsale-rs-xk2pt   1/1     Running   0          2m    10.244.0.10   minikube
flashsale-rs-zt9mn   1/1     Running   0          2m    10.244.0.11   minikube
```
All 5 pods are on the single node (`minikube`).

## Results
- **Pod Distribution** → Each Pod is an identical worker. Scaling creates clones.
- **Resiliency** → When a Pod is deleted, the ReplicaSet automatically creates a new one.
- **Efficiency** → Add Pods on demand spike, remove when demand is low.
- **Real-world** → Same technique used by Netflix, YouTube, Swiggy during peak hours.

## Review Questions

**Q1. What is the initial number of replicas?** → 3

**Q2. How many pods run after applying the ReplicaSet?** → 3

**Q3. What happens when you scale to 5?** → Kubernetes creates 2 additional pods to reach the desired state of 5.

**Q4. What happens when you delete a pod?** → Kubernetes automatically creates a replacement pod to maintain 5 replicas.

**Q5. How does Kubernetes maintain desired replicas?** → It continuously monitors running pods vs desired count and reconciles any difference.

**Q6. How many nodes are running?** → 1

**Q7. Where are pods running?** → All 5 pods on the single minikube node.
