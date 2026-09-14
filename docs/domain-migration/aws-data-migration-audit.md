# Audit and Verify AWS Data Migration Before AWS Decommissioning

**Status:** Active Audit / Decommissioning Blocker  
**Discovered / Initiated:** September 14, 2026  
**Related Master Ticket:** [Ticket #13 in migration-tickets.md](file:///D:/Projects/Upcloud/migration-tickets.md#ticket-13-audit-and-verify-aws-data-migration-before-aws-decommissioning)  
**Parent Index:** [README.md](file:///D:/Projects/Upcloud/README.md)

---

## 1. Background & Decommissioning Objective

The application and domain infrastructure has been successfully migrated to UpCloud Managed Kubernetes (`UKS`), and the standard Kubernetes/Ingress/TLS domain cutover to `mlthrive.com` is now largely complete across all microservices.

However, the legacy AWS account still contains an extensive inventory of **S3 buckets (29 buckets)** and **RDS database instances (8 databases)**.

Before the AWS account can be decommissioned and shut down, we must verify which AWS resources:
- Are still actively required by running applications;
- Have already been fully migrated to UpCloud;
- Were only copied during an earlier AWS-account-to-AWS-account migration and are historical snapshots;
- Are legacy, test, or system resources that can be safely removed;
- Still need to be migrated to UpCloud Object Storage or Managed Databases.

> [!CAUTION]
> **Data Loss Prevention Rule:**  
> No AWS S3 bucket or RDS database instance may be deleted or marked for termination until its status and data disposition have been explicitly confirmed and cross-verified in UpCloud.

---

## 2. Current UpCloud Object Storage Inventory

Endpoint: `6ftru.upcloudobjects.com` (`de-fra1`)

| UpCloud Service | Bucket Name | Approx. Size | Purpose / Workload |
| :--- | :--- | :---: | :--- |
| `static_front_end` | `main-website` | 97.72 MB | Main static website |
| `app-bucket` | `aiwhatif` | 65.31 MB | AI-WhatIf application data |
| `app-bucket` | `surgicsense` | 934.81 MB | SurgicSense application data |
| `nightingale-prod-terraform-state` | `nightingale-prod-tfstate` | 0.07 MB | Terraform state |
| `loki-object-storage` | `loki-chunks` | ~14 GB | Loki log chunks |
| `loki-object-storage` | `loki-admin` | 0 B | Loki administrative state |
| `loki-object-storage` | `loki-ruler` | 0 B | Loki alerting ruler state |

> [!WARNING]
> At the moment, there are **no dedicated UpCloud application buckets** visible for:
> `EchoGame`, `EchoExplore`, `Segmentation3D`, `Cardiomegaly`, `ECG Prediction`, `LifeSaver`, `Bogalusa`, or `HeartClusters`.
> 
> This inventory must be systematically compared against the AWS S3 inventory below.

---

## 3. AWS S3 Inventory

Total buckets identified: **29**

| AWS S3 Bucket Name | Application / Purpose | Current Migration Status & Action |
| :--- | :--- | :--- |
| `healthview-cardiomegaly-data-prod` | Cardiomegaly | **To verify:** Check if backend needs active storage or if dataset is archived. |
| `healthview-ecgprediction-data-prod` | ECG Prediction | **To verify:** Check backend storage requirements. |
| `healthview-echoexplore-videos-prod` | EchoExplore | **UpCloud equivalent not found:** Media migration required if active. |
| `healthview-echogame-videos-prod` | EchoGame | **UpCloud equivalent not found:** Video dataset migration required. |
| `healthview-segmentation3d-assets-prod` | Segmentation3D | **UpCloud equivalent not found:** Verify assets and migrate. |
| `healthview-segmentation3d-models-prod` | Segmentation3D models | 🚨 **Still referenced by application!** Direct dependency in frontend config. Migration required before deletion. |
| `nightingale-aiwhatif-data-prod` | AI-WhatIf | **To verify:** Compare contents and timestamps with UpCloud `aiwhatif` bucket. |
| `nightingale-bogalusa-data-prod` | Bogalusa | **To verify:** Confirm status and target destination. |
| `nightingale-heartcluster-assets-prod` | HeartClusters | **To verify:** Determine if used by running frontend/backend. |
| `nightingale-lifesaver-models-prod` | LifeSaver | **To verify:** Check model loading requirements in backend. |
| `aiwhatif-v2-bucket-migrated` | AI-WhatIf (earlier migration) | **To verify:** Determine whether legacy snapshot or required source data. |
| `echoexplore-videos-migrated` | EchoExplore (earlier migration) | **To verify:** Determine whether legacy snapshot or source data. |
| `echogame-videos-mp4-migrated` | EchoGame (earlier migration) | **Active Origin:** Configured as origin for CloudFront `dc3pdcj61u7t1.cloudfront.net`. Migration required. |
| `segmentation-3d-models-migrated` | Segmentation3D (earlier migration) | **To verify:** Determine whether duplicate of `healthview-segmentation3d-models-prod`. |
| `cardiomegaly-counterfactuals-migrated` | Cardiomegaly | **To verify:** Determine whether still required by model inference. |
| `cardiomegaly-frontend-migrated1` | Cardiomegaly frontend | **To verify:** Likely legacy/static build artifact — verify and confirm deletion. |
| `3d-frontend-deployment-migrated` | Segmentation3D frontend | **To verify:** Likely legacy/static build artifact — verify and confirm deletion. |
| `bogalusa-nightingaleheart-migrated` | Bogalusa | **To verify:** Check relationship to `nightingale-bogalusa-data-prod`. |
| `nightingale-heartclusters-frontend-migrated` | HeartClusters | **To verify:** Likely legacy static frontend — verify. |
| `admin-devsettings-migrated` | Admin / Dev settings | **Unknown:** Owner and purpose unknown. Needs investigation. |
| `nightinblaze1-migrated` | Unknown | **Unknown:** Identify owner and purpose. |
| `ene-testbucket-migrated` | Test bucket | **Legacy candidate:** Likely deletion candidate after verification. |
| `elasticbeanstalk-eu-central-1-654654611936-migrated` | AWS Elastic Beanstalk | **AWS legacy resource:** Safe to delete once AWS workloads are decommissioned. |
| `sagemaker-eu-central-1-654654611936-migrated` | SageMaker artifacts | **AWS legacy resource:** Verify no model dependencies remain. |
| `sagemaker-eu-north-1-654654611936-migrated` | SageMaker artifacts | **AWS legacy resource:** Verify no model dependencies remain. |
| `sagemaker-studio-654654611936-qi3rk2cq8x-migrated` | SageMaker Studio | **AWS legacy resource:** Confirm user notebooks/data are backed up. |
| `spotify-backend-builds-654654611936-eu-west-1-migrated` | Build artifacts | **To verify:** Check whether build artifacts are still required. |
| `aws-cloudtrail-logs-300763413277-5585bf5e` | AWS CloudTrail audit logs | **System / Audit data:** Compliance/retention decision required before deletion. |
| `nightingale-terraform-state-prod-300763413277` | AWS Terraform state | **Terraform state:** Compare with current UpCloud Terraform state before deletion. |

> [!NOTE]
> Some `*-migrated` buckets originate from an earlier AWS-account-to-AWS-account migration rather than the UpCloud migration. They must **not** automatically be assumed to be present in UpCloud.

---

## 4. Managed Database Services Comparison

UpCloud currently provides two Managed Database clusters:

| UpCloud Managed Service | Engine | Operational State |
| :--- | :--- | :---: |
| `postgres-dbs` | PostgreSQL | Running |
| `mysql-dbs` | MySQL | Running |

AWS RDS currently contains the following 8 instances:

| AWS RDS Instance Identifier | Engine | Associated Application | Expected UpCloud Target | Verification State |
| :--- | :--- | :--- | :--- | :---: |
| `healthmap-adaptatutor-db-prod` | PostgreSQL | AdaptaTutor | `postgres-dbs` | ⚠️ Backend had placeholder credentials (Ticket #12). Database data must be confirmed. |
| `healthmap-heartsayings-db-prod` | MySQL | Heart Sayings | `mysql-dbs` | 🔍 Verify logical DB name, tables, and rows in `mysql-dbs`. |
| `healthmap-pollutionmap-db-prod` | MySQL | PollutionMap | `mysql-dbs` | 🔍 Verify logical DB name, tables, and rows in `mysql-dbs`. |
| `healthmap-worldhealthmap-db-prod` | MySQL | WorldHealthMap | `mysql-dbs` | 🔍 Verify logical DB name, tables, and rows in `mysql-dbs`. |
| `healthview-cardiomegaly-db-prod` | MySQL | Cardiomegaly | `mysql-dbs` | 🔍 Verify logical DB name, tables, and rows in `mysql-dbs`. |
| `nightingale-harmoniahealth-db-prod` | PostgreSQL | HarmoniaHealth | `postgres-dbs` | 🔍 Verify logical DB name, tables, and rows in `postgres-dbs`. |
| `nightingale-lifesaver-db-prod` | PostgreSQL | LifeSaver | `postgres-dbs` | 🔍 Verify logical DB name, tables, and rows in `postgres-dbs`. |
| `surgicsense-db` | PostgreSQL | SurgicSense | `postgres-dbs` | 🔍 Verify logical DB name, tables, and rows in `postgres-dbs`. |

> [!IMPORTANT]
> The UpCloud Managed Database services are running, but the individual logical databases, schemas, user grants, and row counts have not yet been fully audited against AWS RDS.

---

## 5. Known Active AWS Dependencies in Cluster

Several workloads running in the UpCloud Kubernetes cluster still actively depend on AWS storage and configuration:

1. **Segmentation3D AI Models:**
   * Referenced directly in [apps/healthview-segmentation3d/frontend-configmap.yaml](file:///D:/Projects/Upcloud/gitops-infra/apps/healthview-segmentation3d/frontend-configmap.yaml):
     ```text
     https://healthview-segmentation3d-models-prod.s3.eu-central-1.amazonaws.com/models
     ```
   * Cannot be decommissioned until models are migrated to UpCloud Object Storage and the ConfigMap is updated (see [Ticket #9](file:///D:/Projects/Upcloud/migration-tickets.md#ticket-9-segmentation3d-models-storage-migration-aws---approved-storage)).

2. **EchoExplore Backend AWS Secret:**
   * The EchoExplore backend deployment mounts an AWS/S3 credential Secret.
   * Requires auditing to determine if it actively uploads/downloads ultrasound media to/from AWS S3 (see [Ticket #10](file:///D:/Projects/Upcloud/migration-tickets.md#ticket-10-echoexplore-media-storage-dependency-audit-and-migration)).

3. **EchoGame CloudFront & S3 Origin:**
   * EchoGame frontend references legacy CloudFront distribution `dc3pdcj61u7t1.cloudfront.net` (origin `echogame-videos-mp4.s3.eu-central-1.amazonaws.com`) via `NEXT_PUBLIC_S3_BASE_URL` and `CDN_BASE_URL`.
   * Video dataset must be copied to UpCloud Object Storage and given a public or CDN endpoint before decommissioning (see [Ticket #8](file:///D:/Projects/Upcloud/migration-tickets.md#ticket-8-migrate-echogame-media-storage-and-cdn-from-aws-to-upcloud)).

---

## 6. Required Investigation Workflow

For each AWS S3 bucket and RDS database instance:

1. **Identify Owner & Application:** Determine the specific owning application, service, or system.
2. **Determine Active Usage:** Inspect application code, GitOps manifests, running pods, and access logs to verify whether the resource is actively accessed.
3. **Locate Target in UpCloud:** Identify the corresponding UpCloud Object Storage bucket or Managed Database.
4. **Compare Data Content:** Compare file trees, object counts, total byte size, and database row counts between AWS and UpCloud.
5. **Verify Data Integrity:** Confirm that the UpCloud copy is complete, intact, and sufficiently up to date.
6. **Verify Traffic Cutover:** Confirm that the running application exclusively connects to UpCloud and no longer queries AWS.
7. **Assign Status Tag:** Mark each AWS resource with one of five explicit classifications:
   * `MIGRATED / VERIFIED` — Complete in UpCloud; application verified; safe to decommission.
   * `MIGRATION REQUIRED` — Actively needed by application but not yet migrated to UpCloud.
   * `LEGACY` — Historical or abandoned resource; no longer used; candidate for deletion after snapshot/retention approval.
   * `SYSTEM / RETENTION REQUIRED` — AWS audit, CloudTrail, or state data requiring archival before deletion.
   * `UNKNOWN / OWNER REQUIRED` — Ownership unclear; requires outreach to project leads before any action.
8. **Create Tracking Tickets:** Generate discrete sub-tickets for all resources marked `MIGRATION REQUIRED`.

---

## 7. Acceptance Criteria for AWS Decommissioning

The AWS account can only be approved for final shutdown and decommissioning when:

- [ ] Every S3 bucket (all 29) has an explicit disposition tag approved by stakeholders.
- [ ] Every RDS database instance (all 8) has been audited and its contents verified in UpCloud `postgres-dbs` or `mysql-dbs`.
- [ ] All active application references to `amazonaws.com` and `cloudfront.net` are eliminated from GitOps manifests and container builds.
- [ ] All required application media and models are served from UpCloud Object Storage.
- [ ] Audit logs and Terraform state files have been archived according to organizational compliance policy.
- [ ] No AWS S3 bucket or RDS instance receives production traffic.
