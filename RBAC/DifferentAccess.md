# JWT Tokens in Kubernetes & OIDC (Complete Comparison)

This document explains how JWT tokens look and behave in three important scenarios:

1. **User Authentication (Human → Kubernetes via OIDC)**
2. **Pod Accessing Kubernetes API (ServiceAccount → RBAC)**
3. **Pod Accessing Azure Resources (Workload Identity)**

---

## 1. User Authentication (OIDC Token from External IdP)

This token is issued by an external Identity Provider such as Azure AD.

```json
{
  "iss": "https://login.microsoftonline.com/<tenant-id>/v2.0",
  "sub": "8f9c1e8c-xxxx-xxxx-xxxx-xxxxxxxx",
  "aud": "kubernetes",
  "exp": 1714243200,
  "iat": 1714239600,
  "name": "Nitin Rohit",
  "email": "nitin@example.com",
  "groups": [
    "platform-admins",
    "dev-team"
  ]
}
```

### Key Points

- **Issuer (`iss`)**: External IdP (Azure AD)
- **Subject (`sub`)**: User identity
- **Audience (`aud`)**: Kubernetes client
- **Extra Claims**: email and groups (used by RBAC)

Kubernetes extracts:

```text
User: nitin@example.com
Groups: platform-admins, dev-team
```

---

## 2. Pod Accessing Kubernetes API (ServiceAccount Token)

This token is issued by Kubernetes itself.

```json
{
  "iss": "https://<cluster-oidc-issuer>",
  "sub": "system:serviceaccount:dev:app-sa",
  "aud": ["https://kubernetes.default.svc"],
  "exp": 1714243200,
  "iat": 1714239600,
  "kubernetes.io": {
    "namespace": "dev",
    "serviceaccount": {
      "name": "app-sa",
      "uid": "c3f5f5d0-xxxx"
    },
    "pod": {
      "name": "app-pod-abc123",
      "uid": "9e12a34b-xxxx"
    }
  }
}
```

### Key Points

- **Issuer (`iss`)**: Kubernetes
- **Subject (`sub`)**: ServiceAccount identity
- **Audience (`aud`)**: Kubernetes API
- **Extra Claims**: Pod and ServiceAccount metadata

Kubernetes extracts:

```text
User: system:serviceaccount:dev:app-sa
Groups:
- system:serviceaccounts
- system:serviceaccounts:dev
```

---

## 3. Pod Accessing Azure Resources (Workload Identity)

This is the same type of token as above, but with a different audience.

```json
{
  "iss": "https://<cluster-oidc-issuer>",
  "sub": "system:serviceaccount:dev:app-sa",
  "aud": ["api://AzureADTokenExchange"],
  "exp": 1714243200,
  "iat": 1714239600
}
```

### Key Points

- **Issuer (`iss`)**: Kubernetes
- **Subject (`sub`)**: Same ServiceAccount identity
- **Audience (`aud`)**: Azure token exchange endpoint
- **Purpose**: Authentication to Azure AD

Azure AD validates:

- Issuer (must trust cluster OIDC)
- Subject (must match federated identity)
- Audience (must match expected value)

---

## Side-by-Side Comparison

| Field | Human OIDC | Pod → Kubernetes | Pod → Azure |
|------|-----------|------------------|------------|
| Issuer (`iss`) | Azure AD | Kubernetes | Kubernetes |
| Subject (`sub`) | User ID | ServiceAccount | ServiceAccount |
| Audience (`aud`) | Kubernetes | Kubernetes API | Azure |
| Extra Claims | email, groups | pod metadata | minimal |
| Verified By | kube-apiserver | kube-apiserver | Azure AD |
| Purpose | Login | RBAC | Cloud access |

---

## Key Insights

- All three tokens are **JWT and OIDC-compliant**
- The **issuer and audience determine the purpose**
- The same ServiceAccount identity can be used for both:
  - Kubernetes API access
  - Azure resource access

---

## Mental Model

```
Same JWT structure
        ↓
Different issuer + audience
        ↓
Different system verifies it
        ↓
Different purpose achieved
```

---

## One-Line Summary

**All three scenarios use JWT tokens with similar structure, but differ in issuer and audience—external IdP for users, Kubernetes for pods—with the audience deciding whether the token is used for Kubernetes RBAC or external systems like Azure.**
