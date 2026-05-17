# Kubernetes PV, PVC, and Pod Connection

## 📌 Overview

In Kubernetes, Persistent Volumes (PV) and Persistent Volume Claims (PVC) are used to provide persistent storage to Pods. This ensures data is not lost even if a Pod restarts or is recreated.

---

# 🧠 Core Concepts

## ✅ PersistentVolume (PV)
- Represents actual storage in the cluster
- Created by admin or dynamically provisioned
- Example storage: cloud disks, NFS, or local disk (hostPath)

---

## ✅ PersistentVolumeClaim (PVC)
- A request for storage by a user/application
- Binds to a matching PV based on size and access mode

---

## ✅ Pod
- Uses PVC to access storage
- Does not directly interact with PV

---

# 🔄 Flow

```
PV → PVC → Pod
```

- PV provides storage
- PVC requests storage
- Pod consumes storage via PVC

---

# 🔧 YAML Configuration

## 1️⃣ PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

---

## 2️⃣ PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

---

## 3️⃣ Pod using PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: my-storage
      mountPath: /data
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

---

# 🧪 Practical Demonstration

## ✅ Step 1: Apply PV
```bash
kubectl apply -f pv.yaml
```

## ✅ Step 2: Apply PVC
```bash
kubectl apply -f pvc.yaml
```

## ✅ Step 3: Apply Pod
```bash
kubectl apply -f pod.yaml
```

---

## ✅ Step 4: Create File Inside Pod
```bash
kubectl exec -it busybox-pod -- sh
```

Inside container:
```sh
echo "hello world" > /data/test.txt
cat /data/test.txt
```

---

## ✅ Step 5: Delete Pod
```bash
kubectl delete pod busybox-pod
```

---

## ✅ Step 6: Recreate Pod
```bash
kubectl apply -f pod.yaml
```

---

## ✅ Step 7: Verify Data Persistence
```bash
kubectl exec -it busybox-pod -- cat /data/test.txt
```

✅ Expected Output:
```
hello world
```

---

# 🎯 Conclusion

- Data persists even after Pod deletion ✅
- PV provides storage
- PVC binds to PV
- Pod uses PVC

---

# 💡 Key Takeaway

> PV is the storage, PVC is the request, and Pod is the consumer.
