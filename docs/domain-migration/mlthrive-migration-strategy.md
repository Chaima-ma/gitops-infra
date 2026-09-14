# Service Migration Strategy & Implementation Plan (`mlthrive.com`)

**Approval Date:** September 14, 2026  
**Context:** Migrating production Kubernetes services from the expired legacy domain `*.nightingaleheart.com` (NXDOMAIN) to the production domain `*.mlthrive.com`.  
**Root Domain Status:** `https://mlthrive.com/` is actively hosted on UpCloud Managed Object Storage.

---

## 1. Architectural Constraints & Critical Discoveries

### ⚠️ Constraint №1: DNS Collision on `_acme-challenge.mlthrive.com` (RFC 1034)
* **Cloudflare State:**  
  The root domain `mlthrive.com` already utilizes:
  ```text
  _acme-challenge.mlthrive.com  CNAME  <endpoint-upcloud-storage>
  ```
  This record is required by UpCloud Object Storage to automatically issue and renew the root website's SSL/TLS certificate.
* **Risk & Collision:**  
  Attempting to issue a wildcard certificate `*.mlthrive.com` via Cert-Manager DNS-01 (Cloudflare) causes cert-manager to attempt creating:
  ```text
  _acme-challenge.mlthrive.com  TXT  <challenge-token>
  ```
  Under **RFC 1034 and RFC 2181**, a `CNAME` record **cannot coexist** with any other record type (including `TXT`) at the same FQDN.
* **Consequence:**  
  Issuing `*.mlthrive.com` via DNS-01 would either fail at the Cloudflare API layer or overwrite the existing CNAME, **breaking TLS for the main website on Object Storage**.
* **Architectural Solution:**  
  Migrate all Kubernetes services **via NGINX Ingress using HTTP-01 validation** (`letsencrypt-prod`). This approach completely avoids touching `_acme-challenge` DNS records in Cloudflare.

---

### ⚠️ Constraint №2: Wildcard Certificate Scope Limitation (RFC 6125) for Multi-Level Subdomains
* **Legacy Cilium Gateway Behavior:**  
  The certificate `wildcard-nightingale-certificate` covered only `*.nightingaleheart.com`.
* **RFC 6125 Standard Limitation:**  
  The wildcard character `*` matches strictly **one** DNS label:
  * `aiwhatif.nightingaleheart.com` — ✅ Valid (1 label).
  * `api.aiwhatif.nightingaleheart.com` — ❌ **Invalid** (2 labels, browser reports `ERR_CERT_COMMON_NAME_INVALID`).
  * `api-python.aiwhatif.nightingaleheart.com` — ❌ **Invalid** (2 labels).
* **Consequence:**  
  The only fully functional path for multi-level API endpoints was **NGINX Ingress**, where certificates were issued via HTTP-01 explicitly listing each SAN.
* **Solution:**  
  For services with multi-level APIs, issue certificates with explicit host lists through NGINX Ingress or provision dedicated subdomain wildcards (`*.aiwhatif.mlthrive.com`).

---

### ⚠️ Constraint №3: Cloudflare Wildcard Points to Object Storage
* Cloudflare currently resolves:  
  `*.mlthrive.com` → `6ftru.upcloudobjects.com` (UpCloud Object Storage).
* Any Kubernetes subdomain without an explicit record will be routed to Object Storage and return HTTP 404.
* **Solution:**  
  Every Kubernetes service must have an **explicit CNAME record** targeting the NGINX Load Balancer:
  ```text
  <service>.mlthrive.com  CNAME  lb-0a9b1179d97749e8914609fb8f972856-1.upcloudlb.com  (DNS only)
  ```

---

## 2. Architecture and Resolution for `ai-whatif`

Originally, `ai-whatif` was in an **OutOfSync / Degraded** status under ArgoCD, and was thought to require Cilium Gateway API for routing.

1. **Historical Root Cause of ArgoCD OutOfSync:**  
   The single out-of-sync resource was `HTTPRoute/backendr-aiwhatif-httproute`. An indentation error existed in `apps/ai-whatif/backendR-httpRoute.yaml`:
   ```yaml
   # HISTORICAL DEFECTIVE SYNTAX:
     rules:
     - matches:
       - path:
         type: PathPrefix
         value: /
   ```
   Because `type` and `value` were placed at the same indentation level as `path:`, YAML parsed the node as `{"path": null, "type": "PathPrefix", "value": "/"}`.

2. **Implemented Resolution (Consolidation onto NGINX Ingress):**  
   Rather than preserving duplicate routing through Cilium Gateway API or repairing the defective HTTPRoute, **all AI-WhatIf traffic was permanently consolidated onto the standard NGINX Ingress path**.
   
   Cilium Gateway is **no longer required** for AI-WhatIf. All three legacy HTTPRoutes (`frontend-httpRoute.yaml`, `backendPy-httpRoute.yaml`, `backendR-httpRoute.yaml`) were deleted from GitOps and pruned from the cluster by ArgoCD.

3. **Final Implemented Architecture:**  
   ```text
   aiwhatif.mlthrive.com
   healthyheart.mlthrive.com
   api.aiwhatif.mlthrive.com
   api-python.aiwhatif.mlthrive.com
           ↓
   NGINX Ingress (lb-0a9b1179... 212.147.228.214)
   aiwhatif-mlthrive-tls (HTTP-01 Let's Encrypt)
   ```

   * **Manifest:** [apps/ai-whatif/ingress.yaml](file:///d:/Projects/Upcloud/gitops-infra/apps/ai-whatif/ingress.yaml)
   * **TLS Secret:** `aiwhatif-mlthrive-tls` (all 4 SANs issued cleanly via HTTP-01)
   * **Current ArgoCD State:** `Synced / Healthy`
   * **Cluster HTTPRoutes in `ai-whatif`:** None (pruned)

---

## 3. Safe Migration Workflow (per Service)

Each service migrates **individually** without mass search-and-replace scripts, adhering to a 7-step lifecycle:

```mermaid
flowchart LR
    S1["1. DNS in Cloudflare<br/>(CNAME -> NGINX LB)"] --> S2["2. Edit Ingress in Git<br/>(New host + dedicated TLS Secret)"]
    S2 --> S3["3. Push to master<br/>(ArgoCD Sync)"]
    S3 --> S4["4. Cert-Manager<br/>(HTTP-01 TLS Issuance)"]
    S4 --> S5["5. HTTPS Testing<br/>(curl / browser)"]
    S5 --> S6["6. Update ConfigMap<br/>(Hardcoded URLs)"]
    S6 --> S7["7. Remove legacy hosts<br/>(Clean-up)"]
```

### Safe Ingress Configuration Pattern (Dual Host + Isolated Secrets):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <service>-ingress
  namespace: <service>
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    # Legacy block (preserved until cutover verification completes)
    - hosts:
        - <service>.nightingaleheart.com
      secretName: <service>-tls
    # New isolated block for target domain
    - hosts:
        - <service>.mlthrive.com
      secretName: <service>-mlthrive-tls
  rules:
    - host: <service>.nightingaleheart.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: <service-backend>
                port:
                  number: 80
    # New rule for mlthrive
    - host: <service>.mlthrive.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: <service-backend>
                port:
                  number: 80
```

---

## 4. Pilot Execution: `worldhealthmap`

`worldhealthmap` served as **Pilot №1**:
* ArgoCD status: **Synced + Healthy**;
* Architecture: Pure SPA frontend without complex inter-service database dependencies;
* Operates exclusively via NGINX Ingress (isolated from Cilium Gateway API).

### Pilot Verification Checklist:

- [x] **Step 1. DNS (Cloudflare):**
  * Create `worldhealthmap.mlthrive.com` CNAME `lb-0a9b1179d97749e8914609fb8f972856-1.upcloudlb.com` (DNS only).
- [x] **Step 2. Git (`apps/healthmap-worldhealthmap/ingress.yaml`):**
  * Add host `worldhealthmap.mlthrive.com` and secret `worldhealthmap-mlthrive-tls`.
- [x] **Step 3. Git Push & ArgoCD:**
  * Commit and push to `master`. Reconcile in ArgoCD.
- [x] **Step 4. Certificate Validation:**
  ```powershell
  kubectl -n worldhealthmap get certificate,ingress
  ```
- [x] **Step 5. Endpoint Validation:**
  ```powershell
  curl.exe -Iv https://worldhealthmap.mlthrive.com
  ```

---

## 5. Migration Waves Overview

| Wave | Services | Complexity | Key Requirements |
| :---: | :--- | :---: | :--- |
| **Wave 1 (Simple UI)** | `worldhealthmap`, `healthmap-heart-sayings`, `heartaware` | Low | Single hosts, static/SPA frontends |
| **Wave 2 (Frontend + Backend)** | `causal-modeling`, `ecgprediction`, `pollutionmap`, `cardiomegaly-cnn`, `echoexplore`, `harmoniahealth` | Medium | Dual hosts (UI + API), HTTP-01 certificates |
| **Wave 3 (ConfigMap / ENV Dependencies)**| `healthview-surgicsense`, `healthview-segmentation3d`, `healthview-echogame` | Medium-High | Requires synchronized updates to ConfigMap/Deployments |
| **Wave 4 (Complex Services)** | `ai-whatif` (Completed: NGINX Ingress), `healthmap-adaptatutor` | High | AI-WhatIf verified on NGINX Ingress; legacy backend recovery for `adaptatutor` (Ticket #12) |
| **Wave 5 (Infrastructure)** | `cilium-apiGateway`, `cluster-issuer` | Medium | Contact email updates and Gateway API support alignment |
