# Kubernetes ConfigMap, Secret, and SecretProviderClass (SPC)

## 📌 Overview

This document explains Kubernetes **ConfigMap**, **Secret**, and **SecretProviderClass (SPC)**, their purpose, usage, and practical examples.

---

# 🚀 ConfigMap

## 🧠 What is ConfigMap?

A ConfigMap is used to store **non-sensitive configuration data** in key-value format and inject it into Pods.

---

## ✅ Why use ConfigMap?

- Avoid hardcoding configs inside application code
- Modify configuration without rebuilding images
- Separate configuration from application logic

---

## 🔬 Example ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: debug
```

---

## 🔄 Using ConfigMap

### 1. As Environment Variables

```yaml
env:
- name: APP_ENV
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_ENV
```

---

### 2. As Volume

```yaml
volumeMounts:
- name: config-volume
  mountPath: /etc/config

volumes:
- name: config-volume
  configMap:
    name: app-config
```

---

# 🔐 Secret

## 🧠 What is Secret?

A Secret is used to store **sensitive data** like passwords, API keys, and tokens.

---

## ✅ Why use Secret?

- Keeps sensitive data separate from code
- More secure than ConfigMap

---

## 🔬 Example Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  PASSWORD: bXlwYXNzd29yZA==
```

---

## 🔄 Using Secret

### 1. As Environment Variables

```yaml
env:
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: PASSWORD
```

---

### 2. As Volume

```yaml
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets

volumes:
- name: secret-volume
  secret:
    secretName: app-secret
```

---

# 🔥 ConfigMap vs Secret

- ConfigMap → Non-sensitive data
- Secret → Sensitive data (base64 encoded)

---

# 🔄 Practical: ConfigMap + Secret

## Step 1: Create ConfigMap

```bash
kubectl create configmap app-config --from-literal=APP_ENV=dev
```

## Step 2: Create Secret

```bash
kubectl create secret generic app-secret --from-literal=PASSWORD=mypassword
```

## Step 3: Pod YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sleep", "3600"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: PASSWORD
```

---

## Step 4: Verify

```bash
kubectl exec -it test-pod -- env | grep APP_ENV
kubectl exec -it test-pod -- env | grep PASSWORD
```

---

# 🔐 SecretProviderClass (SPC)

## 🧠 What is SPC?

SPC is used with **Secrets Store CSI Driver** to fetch secrets from external secret providers like:

- Azure Key Vault
- AWS Secrets Manager
- GCP Secret Manager

This part has been explained in the RBAC section more detailedly as it involves giving permissions and connecting kubernetes resource to external resource.

---

## 🔄 Flow

```
Pod → CSI Driver → Provider Plugin → External Secret Store
```

---

## 🔬 Example SPC (Azure example)

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: azure-spc
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"
    keyvaultName: "my-keyvault"
    objects: |
      array:
        - |
          objectName: secret1
          objectType: secret
    tenantId: "<tenant-id>"
```

---

## 🔄 Pod using SPC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spc-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: secrets-store-inline
      mountPath: /mnt/secrets
  volumes:
  - name: secrets-store-inline
    csi:
      driver: secrets-store.csi.k8s.io
      readOnly: true
      volumeAttributes:
        secretProviderClass: "azure-spc"
```

---

## ✅ Practical Steps (SPC)

1. Install Secrets Store CSI Driver
2. Create SecretProviderClass
3. Deploy Pod using SPC
4. Verify secrets inside Pod

```bash
kubectl exec -it spc-pod -- ls /mnt/secrets
```

---

# 🎯 Key Takeaways

- ConfigMap → configuration data
- Secret → sensitive data
- SPC → external secret integration
- All help decouple config and secrets from application

---

# 💡 One-line Summary

> ConfigMap and Secret manage application configuration and sensitive data, while SPC enables secure integration with external secret stores using CSI drivers.
