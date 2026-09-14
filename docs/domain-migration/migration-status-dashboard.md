# Service Migration Status Dashboard (`mlthrive.com`)

**Last Updated:** September 14, 2026  
**Primary Ingress Load Balancer:** `lb-0a9b1179d97749e8914609fb8f972856-1.upcloudlb.com` (`212.147.228.214`)  
**Overall Progress:** `[██████████████░] 93% (14 of 15 services migrated on K8s Ingress/TLS level)`

---

## 1. Summary Status Table for All Services

| № | Service / Repository | Wave | Target Host(s) | DNS (Cloudflare) | Ingress in Git | TLS (Cert-Manager) | HTTP Test | Final Status |
| :-: | :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| 1 | **World Health Map** | 1 | `worldhealthmap.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟢 **Complete** |
| 2 | **Health Sayings** | 1 | `healthmap.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟢 **Complete** |
| 3 | **Life Saver** | 1 | `lifesaver.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟢 **Complete** |
| 4 | **Heart Aware** | 1 | `heartaware.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `307 Redirect` | 🟡 **Ticket #7** (Auth0 callback) |
| 5 | **Causal Modeling** | 2 | `causal-modeling.mlthrive.com`<br/>`api-causal-modeling.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟡 **Ticket #1** (Frontend hardcode) |
| 6 | **ECG Prediction** | 2 | `ecgprediction.mlthrive.com`<br/>`api.ecgprediction.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟢 **Complete** |
| 7 | **Pollution Map** | 2 | `pollutionmap.mlthrive.com`<br/>`api.pollutionmap.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` (API) | 🟡 **Ticket #2** (Frontend hardcode) |
| 8 | **EchoExplore** | 2 | `echoexplore.mlthrive.com`<br/>`api-echoexplore.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` (`/predict`) | 🟢 **Complete** (see Ticket #4 audit) |
| 9 | **Cardiomegaly CNN** | 2 | `cardiomegaly-cnn.mlthrive.com`<br/>`api-cardiomegaly.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟡 **Ticket #7** (Auth0 callback) |
| 10 | **Harmonia Health** | 2 | `harmoniahealth.mlthrive.com`<br/>`api-harmoniahealth.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` (Both) | 🟡 **Ticket #5** (Frontend old API URL) |
| 11 | **EchoGame** | 3 | `echogame.mlthrive.com`<br/>`api-echogame.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` (Domain/API) | 🟡 **Ticket #3 & #8** (Frontend hardcode + Media AWS) |
| 12 | **3D Segmentation** | 3 | `segmentation.mlthrive.com`<br/>`segmentation-api.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | Frontend `200` | 🟡 **Ticket #9 & #11** (S3 Models + Cilium CNI 503) |
| 13 | **SurgicSense** | 3 | `surgicsense.mlthrive.com`<br/>`api.surgicsense.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` | 🟡 **Ticket #6 & #7** (Frontend hardcode + Auth0) |
| 14 | **Adaptatutor** | 4 | `adaptatutor.mlthrive.com`<br/>`api-adaptatutor.mlthrive.com` | ✅ OK | ✅ Synced | ✅ `READY: True` | `200 OK` (Frontend) | 🟢 **Infra Ready** / 🔴 **Ticket #12** (Backend/DB migration) |
| 15 | **AI What-If** | 4 | `aiwhatif.mlthrive.com` + 3 API hosts | ⏳ Pending CNAME | ⏳ Queued | ⏳ Queued | — | 🔴 **Gateway API / Cilium Support** (Ticket #11) |

---

## 2. Done Log (Pilot Verification)

### 🟢 1. `worldhealthmap.mlthrive.com` (Pilot)
* **Ingress:** [apps/healthmap-worldhealthmap/ingress.yaml](file:///d:/Projects/Upcloud/gitops-infra/apps/healthmap-worldhealthmap/ingress.yaml)
* **Secret:** `worldhealthmap-mlthrive-tls`
* **Verification Result:**  
  `https://worldhealthmap.mlthrive.com/` → **`200 OK`**.  
  Confirmed the complete path: Cloudflare → UpCloud LB → NGINX → Cert-Manager → Pod.

### 🟢 2. `healthmap.mlthrive.com` (`healthmap-heart-sayings`)
* **Ingress:** [apps/healthmap-heart-sayings/ingress.yaml](file:///d:/Projects/Upcloud/gitops-infra/apps/healthmap-heart-sayings/ingress.yaml)
* **Secret:** `healthmap-heart-sayings-mlthrive-tls`
* **Backend:** `healthmap-heart-sayings:5001`
* **Verification Result:**  
  `https://healthmap.mlthrive.com/` → `302` → `https://healthmap.mlthrive.com/heart-sayings` → **`200 OK`**.

### 🟢 3. `lifesaver.mlthrive.com` (`nightingale-lifesaver`)
* **Ingress:** [apps/nightingale-lifesaver/ingress.yaml](file:///d:/Projects/Upcloud/gitops-infra/apps/nightingale-lifesaver/ingress.yaml)
* **Secret:** `lifesaver-mlthrive-tls`
* **Routing:** Supports all 3 paths: `/api` (port 3000), `/webhooks` (port 5005), `/` (port 80).
* **Verification Result:**  
  `https://lifesaver.mlthrive.com/` → **`200 OK`**.

### 🟡 4. `heartaware.mlthrive.com` (`nightingale-heartaware`)
* **Ingress:** [apps/nightingale-heartaware/ingress.yaml](file:///d:/Projects/Upcloud/gitops-infra/apps/nightingale-heartaware/ingress.yaml)
* **Secret:** `heartaware-mlthrive-tls` (issued in 45 seconds, `READY: True`).
* **Verification Result:**  
  Network layer and TLS operational (`307 Temporary Redirect` to `/auth/login`).
* **Blocker Status:** Login awaits addition of `https://heartaware.mlthrive.com/auth/callback` in Auth0 Dashboard (see [pending-actions-and-auth0.md](file:///d:/Projects/Upcloud/pending-actions-and-auth0.md)).

---

## 3. Legacy Domain Cleanup Backlog (`nightingaleheart.com`)

Following confirmation of stable operation on `mlthrive.com`, legacy `nightingaleheart.com` artifacts are cleaned up systematically to prevent configuration drift:

### 1. Ingress Manifests in Git

| Ingress Manifest in Git | Legacy Domain Content | Cleanup Status |
| :--- | :--- | :---: |
| `apps/healthmap-worldhealthmap/ingress.yaml` | `worldhealthmap.nightingaleheart.com` + `worldhealthmap-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthmap-heart-sayings/ingress.yaml` | `healthmap.nightingaleheart.com` + `healthmap-heart-sayings-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/nightingale-lifesaver/ingress.yaml` | `lifesaver.nightingaleheart.com` + `lifesaver-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthview-ecgprediction/ingress.yaml` | `ecgprediction.*`, `api.ecgprediction.*` + `ecgprediction-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthmap-adaptatutor/ingress.yaml` | `adaptatutor.*`, `api-adaptatutor.*` + `adaptatutor-tls`, `api-adaptatutor-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/nightingale-heartaware/ingress.yaml` | `heartaware.nightingaleheart.com` + `heartaware-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/causal-modeling/ingress.yaml` | `causal-modeling.*`, `api-causal-modeling.*` + `causal-modeling-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthmap-pollutionmap/ingress.yaml` | `pollutionmap.*`, `api.pollutionmap.*` + `pollutionmap-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthview-echoexplore/ingress.yaml` | `echoexplore.*`, `api-echoexplore.*` + `echoexplore-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthview-echogame/ingress.yaml` | `echogame.*`, `api-echogame.*` + `echogame-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/nightingale-harmoniahealth/ingress.yaml` | `harmoniahealth.*`, `api-harmoniahealth.*` + `harmoniahealth-tls`, `api-harmoniahealth-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthview-segmentation3d/ingress.yaml` | `segmentation.*`, `segmentation-api.*` + `healthview-segmentation3d-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthview-cardiomegaly-cnn/ingress.yaml` | `cardiomegaly-cnn.*`, `api-cardiomegaly.*` + `cardiomegaly-tls` | ✅ **Cleaned in Git & K8s** |
| `apps/healthview-surgicsense/ingress.yaml` | `surgicsense.*`, `api.surgicsense.*` + `surgicsense-tls` | ✅ **Cleaned in Git & K8s** |

---

### 2. Legacy K8s Objects in Cluster (Certificates & Secrets)
Orphaned certificates and secrets from the legacy domain:

```powershell
# Wave 1
kubectl -n worldhealthmap delete certificate worldhealthmap-tls
kubectl -n worldhealthmap delete secret worldhealthmap-tls

kubectl -n healthmap-heart-sayings delete certificate healthmap-heart-sayings-tls
kubectl -n healthmap-heart-sayings delete secret healthmap-heart-sayings-tls

kubectl -n nightingale-lifesaver delete certificate lifesaver-tls
kubectl -n nightingale-lifesaver delete secret lifesaver-tls

kubectl -n nightingale-heartaware delete certificate heartaware-tls
kubectl -n nightingale-heartaware delete secret heartaware-tls

# Waves 2, 3, and 4
kubectl -n causal-modeling delete certificate causal-modeling-tls
kubectl -n causal-modeling delete secret causal-modeling-tls

kubectl -n ecgprediction delete certificate ecgprediction-tls
kubectl -n ecgprediction delete secret ecgprediction-tls

kubectl -n pollutionmap delete certificate pollutionmap-tls
kubectl -n pollutionmap delete secret pollutionmap-tls

kubectl -n healthview-echoexplore delete certificate echoexplore-tls
kubectl -n healthview-echoexplore delete secret echoexplore-tls

kubectl -n healthview-echogame delete certificate echogame-tls
kubectl -n healthview-echogame delete secret echogame-tls

kubectl -n nightingale-harmoniahealth delete certificate harmoniahealth-tls api-harmoniahealth-tls
kubectl -n nightingale-harmoniahealth delete secret harmoniahealth-tls api-harmoniahealth-tls

kubectl -n healthview-segmentation3d delete certificate healthview-segmentation3d-tls
kubectl -n healthview-segmentation3d delete secret healthview-segmentation3d-tls

kubectl -n healthview-cardiomegaly-cnn delete certificate cardiomegaly-tls
kubectl -n healthview-cardiomegaly-cnn delete secret cardiomegaly-tls

kubectl -n healthview-surgicsense delete certificate surgicsense-tls
kubectl -n healthview-surgicsense delete secret surgicsense-tls

kubectl -n healthmap-adaptatutor delete certificate adaptatutor-tls api-adaptatutor-tls
kubectl -n healthmap-adaptatutor delete secret adaptatutor-tls api-adaptatutor-tls
```

---

### 3. Stale Cert-Manager Artifacts (Challenges & Orders)
Stale challenges left over from failed renewals after domain expiration:
```powershell
# In healthmap-adaptatutor:
kubectl -n healthmap-adaptatutor delete ingress cm-acme-http-solver-qsffv cm-acme-http-solver-5p4md
kubectl -n healthmap-adaptatutor delete certificate adaptatutor-tls api-adaptatutor-tls

# In worldhealthmap:
kubectl -n worldhealthmap delete challenge challenge.acme.cert-manager.io/worldhealthmap-tls-2-4254899132-3382003230
kubectl -n worldhealthmap delete challenge challenge.acme.cert-manager.io/worldhealthmap-tls-3-4254899132-3793126059
```

---

### 4. Hardcoded URLs in ConfigMap & Deployment
Workload files requiring updates in application code / deployments:
* `apps/healthview-echogame/deployment-backend.yaml`: `https://cdn.echogame.nightingaleheart.com` (Ticket #8)
* `apps/healthview-echogame/deployment-frontend.yaml`: `https://api-echogame.nightingaleheart.com`, `cdn.echogame...` (Ticket #3 & #8)
* `apps/healthview-surgicsense/configmap.yaml`: `RESET_PASSWORD_URL` (Updated in GitOps, awaits rollout after Ticket #11)
* `apps/healthview-segmentation3d/frontend-configmap.yaml`: `API_BASE_URL` (Updated in GitOps)
* `apps/healthmap-adaptatutor/secret.example.yaml`: `FRONTEND_URL`, `ALLOWED_ORIGINS` (Ticket #12)

---

### 5. Infrastructure Remnants (Issuer Email & Gateway API)
* `apps/cluster-issuer/cluster-issuer.yaml` (line 7): `email: info@nightingaleheart.com`
* `apps/cluster-issuer/cluster-issuer-wildcard.yaml` (line 8): `email: info@nightingaleheart.com`
* `apps/cilium-apiGateway/api-gateway.yaml`: `hostname: "*.nightingaleheart.com"` + `wildcard-nightingale-certificate`
* `apps/ai-whatif/*-httpRoute.yaml`: hosts `*.nightingaleheart.com`

---

## 4. Documentation References

1. **[migration-roadmap.md](file:///d:/Projects/Upcloud/migration-roadmap.md)** — Standard Operating Procedure (SOP), pilot runbook, and migration wave plan.
2. **[pending-actions-and-auth0.md](file:///d:/Projects/Upcloud/pending-actions-and-auth0.md)** — Action items registry and Auth0 dashboard configuration runbook.
3. **[infrastructure-audit.md](file:///d:/Projects/Upcloud/infrastructure-audit.md)** — Complete audit of the network topology (Dual Load Balancers).
4. **[gitops-infra-analysis.md](file:///d:/Projects/Upcloud/gitops-infra-analysis.md)** — Analysis of the GitOps repository structure and dependencies.
5. **[migration-tickets.md](file:///d:/Projects/Upcloud/migration-tickets.md)** — Actionable tickets registry for development teams (Scopes 1-4).
6. **[domain-migration-cleanup-note-2026-09-14.md](file:///d:/Projects/Upcloud/domain-migration-cleanup-note-2026-09-14.md)** — Summary note on the final cluster audit and cleanup actions.
