## User Authentication via OIDC (Humans)

Kubernetes integrates with external identity providers using **OIDC (OpenID Connect)** to authenticate human users. The process follows these steps:

### 1. Configure kube-apiserver to trust an OIDC provider

The Kubernetes API server is configured with OIDC settings to establish trust with an external Identity Provider (IdP), such as Azure AD, Google, or Okta.

```bash
--oidc-issuer-url=https://login.microsoftonline.com/<tenant-id>/v2.0
--oidc-client-id=<client-id>
--oidc-username-claim=email
--oidc-groups-claim=groups
```

This tells Kubernetes:
- Which OIDC provider to trust
- How to extract user identity (username and groups) from the token

---

### 2. User authenticates with the Identity Provider

When a user runs a `kubectl` command:

```bash
kubectl get pods
```

- `kubectl` redirects the user to the configured OIDC provider
- The user authenticates (e.g., via password, SSO, MFA)
- The IdP issues an **OIDC ID token (JWT)** containing user identity details

---

### 3. kubectl sends the token to kube-apiserver

The OIDC token is included in the API request:

```
Authorization: Bearer <OIDC_ID_TOKEN>
```

This token acts as proof of identity for the request.

---

### 4. kube-apiserver validates the token

The API server performs the following checks:

- Verifies the token signature using the provider’s public keys (JWKS)
- Validates the token issuer (`iss`)
- Confirms the intended audience (`aud`)
- Checks token expiration (`exp`)

If all validations pass, the user is successfully authenticated.

---

### 5. RBAC Authorization

After successful authentication:

- Kubernetes extracts the user identity (username and groups) from the token
- RBAC (Role-Based Access Control) evaluates permissions using:
  - `Role` / `ClusterRole`
  - `RoleBinding` / `ClusterRoleBinding`

Only after this step is the request finally **allowed or denied**.

---

### Key Takeaways

- **OIDC handles authentication (who you are)**
- **RBAC handles authorization (what you can do)**
- The OIDC token is used **only during authentication**, not during RBAC evaluation
