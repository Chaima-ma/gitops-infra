# UpCloud Kubernetes & `mlthrive.com` Migration Documentation Index

This directory contains the complete technical documentation, architectural audits, migration roadmaps, runbooks, and action tickets for the UpCloud Managed Kubernetes cluster and the domain migration from `nightingaleheart.com` to `mlthrive.com`.

---

## 📌 Quick Document Directory

| Document | Purpose / Category | Status |
| :--- | :--- | :---: |
| [migration-status-dashboard.md](file:///D:/Projects/Upcloud/migration-status-dashboard.md) | **Live Migration Dashboard** (All 15 services status & progress meter) | 🟢 Active |
| [migration-roadmap.md](file:///D:/Projects/Upcloud/migration-roadmap.md) | **Migration Roadmap & Standard Operating Procedure (SOP)** | 🟢 Reference |
| [migration-tickets.md](file:///D:/Projects/Upcloud/migration-tickets.md) | **Actionable Tickets Registry** (Scopes 1–4: Dev, Auth0, Data, Infra) | 🟡 Active |
| [domain-migration-cleanup-note-2026-09-14.md](file:///D:/Projects/Upcloud/domain-migration-cleanup-note-2026-09-14.md) | **Final Audit & Ingress Cleanup Summary Note** | 🟢 Complete |
| [aws-data-migration-audit.md](file:///D:/Projects/Upcloud/aws-data-migration-audit.md) | **AWS S3 & RDS Inventory Audit** (29 buckets, 8 RDS, decommissioning plan) | 🟡 Active |
| [mlthrive-migration-strategy.md](file:///D:/Projects/Upcloud/mlthrive-migration-strategy.md) | **Architectural Strategy & DNS Constraint Analysis** | 🟢 Reference |
| [argocd-gitops-audit.md](file:///D:/Projects/Upcloud/argocd-gitops-audit.md) | **GitOps & ArgoCD Applications Audit** (27 apps, tracking, self-heal) | 🟢 Reference |
| [gitops-infra-analysis.md](file:///D:/Projects/Upcloud/gitops-infra-analysis.md) | **`gitops-infra` Repository Deep Dive & CI/CD Pipeline Analysis** | 🟢 Reference |
| [infrastructure-audit.md](file:///D:/Projects/Upcloud/infrastructure-audit.md) | **Cluster Network Infrastructure Audit** (Dual Load Balancers & TLS) | 🟢 Reference |
| [pending-actions-and-auth0.md](file:///D:/Projects/Upcloud/pending-actions-and-auth0.md) | **Auth0 Configuration Runbook & Blocked Secrets Registry** | ⏸️ Pending |
| [upcloud-support-ticket-cilium.md](file:///D:/Projects/Upcloud/upcloud-support-ticket-cilium.md) | **UpCloud Support Ticket** (Cilium Operator Crash & IPAM Failure) | 🔴 Ready to Send |
| [mlthrive-domain-tls-postmortem.md](file:///D:/Projects/Upcloud/mlthrive-domain-tls-postmortem.md) | **Postmortem: `mlthrive.com` DNS & TLS on UpCloud Object Storage** | 🟢 Closed |

---

## 📑 Detailed Guide to Documents

### 1. Active Status & Execution Documents

#### 📊 [migration-status-dashboard.md](file:///D:/Projects/Upcloud/migration-status-dashboard.md)
* **Description:** Live operational dashboard tracking the migration state of all 15 microservices in the cluster.
* **Key Contents:**
  * Master status matrix: Cloudflare DNS, Git Ingress sync, Cert-Manager TLS, and HTTP health check results.
  * Verified Done Log for pilot and wave services.
  * Legacy domain cleanup registry (tracking removal of obsolete `nightingaleheart.com` hosts and secrets).
  * Direct links to associated blocking tickets.

#### 🗺️ [migration-roadmap.md](file:///D:/Projects/Upcloud/migration-roadmap.md)
* **Description:** The master roadmap and execution manual for migrating services to `mlthrive.com`.
* **Key Contents:**
  * End-to-end architectural delivery flow (Cloudflare → UpCloud LB → Ingress-NGINX → Cert-Manager → Pod).
  * Standard Operating Procedure (SOP): 8 mandatory steps per service.
  * Complete reference runbook based on the successful `worldhealthmap` pilot.
  * Schedule and risk classification across Waves 1 through 5.

#### 🎫 [migration-tickets.md](file:///D:/Projects/Upcloud/migration-tickets.md)
* **Description:** Actionable, engineering-ready ticket backlog documenting issues outside the direct scope of infrastructure Ingress/TLS migration.
* **Key Contents:**
  * **Scope 1: Application Code & Build-Time Hardcodes** (Tickets #1–#6: Causal Modeling, PollutionMap, EchoGame, EchoExplore, HarmoniaHealth, SurgicSense).
  * **Scope 2: External Authentication** (Ticket #7: HeartAware, Cardiomegaly CNN, SurgicSense Auth0 settings).
  * **Scope 3: Storage & Data Migration** (Tickets #8–#10: EchoGame CDN, Segmentation3D models in AWS S3, EchoExplore media; Ticket #13: Full AWS S3 & RDS decommission audit).
  * **Scope 4: Cluster Infrastructure & Backend Deployments** (Ticket #11: Resolved Cilium operator / Gateway API TLSRoute compatibility and IPAM recovery; Ticket #12: AdaptaTutor uncompleted backend/database migration).

#### 🗄️ [aws-data-migration-audit.md](file:///D:/Projects/Upcloud/aws-data-migration-audit.md)
* **Description:** Comprehensive inventory, verification workflow, and safety audit for all legacy AWS data resources prior to AWS account decommissioning.
* **Key Contents:**
  * Current UpCloud Object Storage baseline (7 buckets across 5 services).
  * Full AWS S3 inventory (29 buckets classified by owning application and migration status).
  * Managed Database audit matrix (8 AWS RDS instances mapped to UpCloud `postgres-dbs` and `mysql-dbs`).
  * Known active cluster dependencies on AWS (`healthview-segmentation3d-models-prod`, EchoExplore secret, EchoGame CloudFront).
  * 8-step verification workflow and strict decommissioning acceptance criteria.

#### 📝 [domain-migration-cleanup-note-2026-09-14.md](file:///D:/Projects/Upcloud/domain-migration-cleanup-note-2026-09-14.md)
* **Description:** Technical summary note of the final live cluster audit conducted on September 14, 2026.
* **Key Contents:**
  * Confirmation of legacy domain removal across all 15 production NGINX Ingress manifests (including `ai-whatif`).
  * Purging of stale cert-manager ACME challenge resources.
  * Diagnosis and resolution of the SurgicSense ConfigMap `last-applied-configuration` corruption in ArgoCD.
  * Clarification on why historical cluster artifacts generated false-positive legacy domain warnings during raw audits.

---

### 2. Architecture & Technical Audits

#### 📐 [mlthrive-migration-strategy.md](file:///D:/Projects/Upcloud/mlthrive-migration-strategy.md)
* **Description:** Architectural strategy document establishing the technical constraints and design choices for the migration.
* **Key Contents:**
  * Analysis of the RFC 1034 DNS collision on `_acme-challenge.mlthrive.com` with UpCloud Object Storage.
  * Analysis of RFC 6125 wildcard certificate limitations regarding multi-level API subdomains.
  * Rationale for standardizing on HTTP-01 validation via NGINX Ingress.
  * Safe Ingress dual-hosting and secret isolation patterns.

#### 🤖 [argocd-gitops-audit.md](file:///D:/Projects/Upcloud/argocd-gitops-audit.md)
* **Description:** Comprehensive audit of GitOps operations, ArgoCD controllers, and Kubernetes resource ownership.
* **Key Contents:**
  * Inventory and health of all 27 ArgoCD Applications.
  * Ingress ownership matrix (differentiating ArgoCD-managed, dynamic cert-manager solvers, and unmanaged resources).
  * Critical operational rule: impact of `selfHeal: true` and `prune: true` policies on manual CLI modifications.
  * Root cause analysis of out-of-sync and degraded applications (`root`, historical `ai-whatif` resolution, `healthmap-adaptatutor`).

#### 📦 [gitops-infra-analysis.md](file:///D:/Projects/Upcloud/gitops-infra-analysis.md)
* **Description:** In-depth technical breakdown of the `gitops-infra` repository structure and delivery automation.
* **Key Contents:**
  * App of Apps architecture and directory hierarchy.
  * Reusable GitHub Actions CI/CD workflow (`build-and-deploy.yml`) analysis.
  * Catalog of all 15 microservices grouped by functional domain.
  * Detailed inventory of the 25 files in GitOps containing domain dependencies.

#### 🌐 [infrastructure-audit.md](file:///D:/Projects/Upcloud/infrastructure-audit.md)
* **Description:** Baseline network and infrastructure audit of the UpCloud Managed Kubernetes cluster.
* **Key Contents:**
  * Identification of dual UpCloud Managed Load Balancers (LB №1: Ingress-NGINX vs LB №2: Cilium Gateway API).
  * Complete routing table: 20 routed hosts, backend services, and ports.
  * Full Cert-Manager certificate and Issuer inventory.
  * Analysis of parallel routing through Cilium Gateway API.

---

### 3. Runbooks, Support Tickets & Postmortems

#### 🔐 [pending-actions-and-auth0.md](file:///D:/Projects/Upcloud/pending-actions-and-auth0.md)
* **Description:** Runbook and registry for services dependent on external authentication via Auth0 (`nightingale-heart-production.eu.auth0.com`).
* **Key Contents:**
  * Technical explanation of why Kubernetes secrets must not be patched before Auth0 dashboard configuration (preventing `403 Callback URL mismatch`).
  * Step-by-step Auth0 dashboard setup guide for HeartAware, Cardiomegaly CNN, and SurgicSense.
  * Post-Auth0 Kubernetes secret patching commands and verification procedures.
  * List of non-Auth0 services that can proceed independently.

#### 🚨 [upcloud-support-ticket-cilium.md](file:///D:/Projects/Upcloud/upcloud-support-ticket-cilium.md)
* **Description:** Formal, high-severity support ticket ready for submission to UpCloud Managed Kubernetes Support.
* **Key Contents:**
  * Detailed incident description: worker node `medium-fbfpz-5brrp` unable to start workloads due to missing IPv4 PodCIDR allocation.
  * Root cause evidence: `cilium-operator` CrashLoopBackOff caused by Gateway API `TLSRoute` CRD version incompatibility (`v1alpha2` expected by Cilium 1.18.6, but only `v1` served).
  * Concrete remediation requests for UpCloud platform engineers.

#### 🔍 [mlthrive-domain-tls-postmortem.md](file:///D:/Projects/Upcloud/mlthrive-domain-tls-postmortem.md)
* **Description:** Postmortem analysis of the initial `mlthrive.com` DNS and TLS provisioning on UpCloud Managed Object Storage.
* **Key Contents:**
  * Analysis of the initial custom domain verification failure.
  * Resolution via explicit DNS delegation and correct ACME challenge binding.
  * Baseline verification of `https://mlthrive.com/` (HTTP 200 OK).

---

## 🛠️ Key Cluster Infrastructure Coordinates

* **Cluster Name / Type:** UpCloud Managed Kubernetes (`UKS`)
* **Region / Zone:** `de-fra1`
* **Primary Ingress Load Balancer (LB №1):** `lb-0a9b1179d97749e8914609fb8f972856-1.upcloudlb.com` (`212.147.228.214`)
* **Gateway API Load Balancer (LB №2):** `lb-0a26cfb0d0f44bcf921ff7ba806f1739-1.upcloudlb.com` (`212.147.228.215`)
* **Object Storage Endpoint:** `6ftru.upcloudobjects.com` (`94.237.114.244`)
* **GitOps Repository:** `git@github.com:curatimeXai/gitops-infra.git` (`master` branch)
