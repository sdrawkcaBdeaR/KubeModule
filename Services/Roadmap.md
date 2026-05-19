# Kubernetes Services: ClusterIP, NodePort, LoadBalancer

## 📌 Overview

Kubernetes Services provide a stable networking endpoint for Pods, which are ephemeral and can change IPs frequently. Services ensure consistent access to applications.

---

# 🚀 Why Services are Required

- Pods are dynamic → IPs change
- Scaling creates multiple Pods
- Need stable DNS and load balancing

👉 Service solves this by providing:
- Stable IP / DNS
- Load balancing
- Service discovery

---

# 🔄 Types of Services

## 1️⃣ ClusterIP (Default)

### ✅ What it is
- Internal service
- Accessible only inside cluster

### 🔄 Flow
Client Pod → ClusterIP Service → Target Pods

### 🔧 YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### ✅ Use Cases
- Backend APIs
- Microservice communication

---

## 2️⃣ NodePort

### ✅ What it is
- Exposes service via Node's IP and a port

### 🔄 Flow
External → NodeIP:NodePort → Service → Pods

### 🔧 YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007
```

### ✅ Use Cases
- Testing
- Simple external access

---

## 3️⃣ LoadBalancer

### ✅ What it is
- Uses cloud provider to create external load balancer

### 🔄 Flow
Internet → Cloud LB → Node → Pods

### 🔧 YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

### ✅ Use Cases
- Production apps
- Public APIs

---

# 📊 Differences

| Feature | ClusterIP | NodePort | LoadBalancer |
|--------|----------|----------|--------------|
| Accessibility | Internal | External via Node IP | Public IP |
| Use Case | Internal services | Testing | Production |
| Requires Cloud | No | No | Yes |

---

# 🧪 Practical Exercise

## Step 1: Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

## Step 2: Apply Services

Apply all three service YAMLs above.

---

## Step 3: Test Each

### ClusterIP
```bash
kubectl exec -it <pod> -- curl clusterip-service
```

### NodePort
```
http://<NodeIP>:30007
```

### LoadBalancer
```
http://<EXTERNAL-IP>
```

---

# ⚠️ Real Issue Faced (LoadBalancer Not Working)

## 🔍 Problem
- LoadBalancer IP assigned ✅
- Pod and service working ✅
- External access failed ❌

## ✅ Root Cause
- Cloud Network Security Group (NSG) blocked port 80

## ✅ Fix
- Allow inbound traffic on port 80

```
Source: Any
Port: 80
Protocol: TCP
Action: Allow
```

## ✅ Result
- External IP started responding

---

# 🎯 Key Takeaways

- ClusterIP → internal communication
- NodePort → basic external testing
- LoadBalancer → production exposure
- External issues often due to cloud networking, not Kubernetes

---

# 💡 One-line Summary

> Services provide stable access and routing to Pods, while different types control how traffic reaches them.
