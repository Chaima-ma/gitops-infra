# Domain Migration and Cleanup Note — 14 Sep 2026

## Overview

A comprehensive audit of both the GitOps repository (`gitops-infra`) and the live Kubernetes cluster was performed to identify any residual references to `nightingaleheart.com` following the migration to `mlthrive.com`.

---

## 1. NGINX Ingress Services Cleanup

The vast majority of standard Kubernetes / NGINX Ingress services have now been completely cleaned of the legacy domain:

- Legacy `nightingaleheart.com` host rules and TLS blocks have been removed from all Ingress manifests via GitOps.
- New `mlthrive.com` Let's Encrypt certificates have been issued, validated (`READY: True`), and are actively serving traffic.
- The final live cluster audit confirms that **the legacy domain has been eliminated from all production NGINX Ingress routing**.

The only routing infrastructure components still referencing `nightingaleheart.com` are:
- **`ai-whatif`** (HTTPRoutes: `aiwhatif.nightingaleheart.com`, `api.aiwhatif.nightingaleheart.com`, `api-python.aiwhatif.nightingaleheart.com`)
- **`gateway/api-gateway`** (Cilium Gateway API wildcard listener `*.nightingaleheart.com`)

This is an expected, deliberate state: migration of `ai-whatif` and the Cilium Gateway was intentionally deferred to avoid compounding issues while the underlying Cilium operator / Gateway API issue is being addressed by platform support.

---

## 2. Stale Cert-Manager ACME Challenge Cleanup

During the live cluster audit, an initial grep for `nightingaleheart.com` returned numerous objects across multiple namespaces. Investigation showed that these were **stale ACME Challenge objects** left behind by `cert-manager`.

### Findings:
- When `nightingaleheart.com` expired (returning NXDOMAIN), automated Let's Encrypt renewal attempts failed and left pending/failed `Challenge` and `Order` objects in the cluster.
- These objects did not represent active routing paths or traffic dependencies; they were lingering ACME negotiation state tied to certificates that had already been replaced or abandoned.
- Before removal, each namespace was verified to confirm that the corresponding legacy `Certificate` and `Order` objects were no longer active or referenced by live Ingresses.
- Stale challenges were safely deleted across migrated namespaces, while `ai-whatif` and `gateway` namespaces were explicitly excluded from this cleanup.

---

## 3. SurgicSense ConfigMap & ArgoCD Sync Resolution

An isolated issue was identified with `apps/healthview-surgicsense`:

- The Git repository already contained the updated configuration pointing to `surgicsense.mlthrive.com`.
- However, ArgoCD failed to apply the `healthview-surgicsense-config` ConfigMap and remained stuck in an `OutOfSync` state.

### Root Cause & Resolution:
- The live ConfigMap contained a corrupted `kubectl.kubernetes.io/last-applied-configuration` annotation containing broken JSON from a historical manual apply. When ArgoCD attempted to compute the three-way merge patch, it failed with `invalid JSON document`.
- The corrupted annotation was safely removed without modifying the underlying configuration data keys.
- ArgoCD was re-synced successfully. The application is now **Synced and Healthy**, and the live ConfigMap correctly contains `RESET_PASSWORD_URL: "https://surgicsense.mlthrive.com/reset-password"`.
- **Runtime Note:** The running backend pod still holds the older environment value in memory because the ConfigMap is consumed via `envFrom`. Pod restart/rollout has been deferred until cluster networking (Cilium) on node `medium-fbfpz-5brrp` is restored, preventing any scheduling disruption.

---

## 4. EchoGame Build-Time Frontend & CDN Separation

The audit confirmed the exact boundary between infrastructure migration and application dependencies for `healthview-echogame`:

- **Kubernetes Configuration (Complete):** The deployment environment variables in GitOps are already updated to `https://api-echogame.mlthrive.com`.
- **Frontend Bundle Hardcode (Application Issue):** The deployed Next.js frontend bundle still directs client-side API calls to `https://api-echogame.nightingaleheart.com/api/questions`. This is because `NEXT_PUBLIC_*` variables were evaluated and baked into static JavaScript artifacts during the original Docker image build.
- **CDN / Media Storage (Data Migration Issue):** Both frontend and backend deployments still reference `https://cdn.echogame.nightingaleheart.com` for media assets hosted in AWS S3 / CloudFront.
- **Conclusion:** No further Ingress or network configuration is required for EchoGame. Remediation requires:
  1. Rebuilding and redeploying the Next.js frontend container with updated build arguments (Ticket #3).
  2. Migrating the S3/CloudFront media bucket to UpCloud Object Storage (Ticket #8).

---

## 5. Root Cause Analysis: Why Multiple Errors Appeared During Audits

The audit clarified why raw cluster searches initially showed a high volume of `nightingaleheart.com` occurrences:

1. **Not Migration Regressions:** The observed references were not introduced by the `mlthrive.com` migration, but represented accumulated historical state.
2. **Expired ACME State:** Lingering `cert-manager` challenges from failed renewals after domain expiration.
3. **Build-Time Inclusions:** Static frontend bundles embedding dead hostnames prior to containerization.
4. **Historical Annotation Corruption:** Malformed `last-applied-configuration` metadata from previous manual operations.
5. **Platform-Level CNI Degradation:** Independent Cilium operator CrashLoopBackOff and IPAM failure affecting worker node `medium-fbfpz-5brrp`.

Therefore, the presence of `nightingaleheart.com` in raw `kubectl` command output did not indicate that real traffic routing was still using the expired domain. All production routing for standard services has been successfully migrated to `mlthrive.com`.
