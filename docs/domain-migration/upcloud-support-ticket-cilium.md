# UpCloud Support Ticket: Cilium operators in CrashLoopBackOff preventing IPAM allocation on worker node

**Cluster:** Managed Kubernetes (UK8s)  
**Region / Zone:** `de-fra1`  
**Affected Worker Node:** `medium-fbfpz-5brrp` (`10.0.0.23` / `94.237.91.44`)  
**Node Creation Date:** September 07, 2026  
**Severity:** High (Workloads scheduled to new worker node cannot start; pods stuck in ContainerCreating / Pending)  

---

### Subject
Cilium operators are in CrashLoopBackOff, preventing IPAM allocation on a new worker node

---

### Description

Hello UpCloud Support,

We are experiencing an issue with Cilium CNI on our Managed Kubernetes cluster in `de-fra1`.

A new worker node `medium-fbfpz-5brrp` was added on September 7, 2026. Kubernetes reports the node as `Ready`, however Cilium networking has never become functional on this node.

The corresponding `CiliumNode` resource exists, but its IPAM allocation is empty:

```yaml
spec:
  ipam:
    pools: {}
```

While the 5 older worker nodes in the cluster have valid IPv4 CIDR allocations (e.g. `192.168.x.0/24`), `medium-fbfpz-5brrp` has none.

#### Cilium Agent Status
On `medium-fbfpz-5brrp`, the Cilium agent pod (`cilium-jmhx6`) starts, connects to the Kubernetes API server, and discovers all Cilium CRDs, but then remains blocked in a loop waiting for IPAM configuration:

```text
level=warn msg="Waiting for k8s node information" module=agent.controlplane.daemon
error="required IPv4 PodCIDR not available"
```

Because the agent cannot complete initialization, `/var/run/cilium/cilium.sock` is never created.

#### Pod / CNI Impact
Pods scheduled to `medium-fbfpz-5brrp` fail during CNI setup:

```text
FailedCreatePodSandBox:
plugin type="cilium-cni" failed (add):
unable to connect to Cilium agent:
dial unix /var/run/cilium/cilium.sock:
connect: no such file or directory
```

Workloads scheduled to this node (such as `healthview-segmentation3d-backend`, `loki-canary`, etc.) have consequently been stuck in `Pending` / `ContainerCreating` for over 7 days.

#### Root Cause Investigation: Cilium Operator Failure
At the same time, both `cilium-operator` replicas in `kube-system` are in `CrashLoopBackOff`. Their logs show a Gateway API CRD compatibility error:

```text
no matches for kind "TLSRoute" in version "gateway.networking.k8s.io/v1alpha2"
```

Inspection of the cluster CRDs reveals that `tlsroutes.gateway.networking.k8s.io` currently serves only `v1`, while `v1alpha2` and `v1alpha3` are set to `served: false`. The Cilium operator 1.18.6 controller fails during startup because it attempts to watch `v1alpha2`. Because the operator crashes, cluster-pool IPAM allocation does not run and never assigns a PodCIDR to newly joined nodes.

We have intentionally not modified Cilium CRDs, CiliumNode IPAM state, or system components manually to avoid interfering with cluster management.

---

### Request to UpCloud Support

Could you please investigate the Cilium / Gateway API configuration for this managed cluster and:

1. Resolve the `TLSRoute` version mismatch so that `cilium-operator` can start cleanly and remain healthy.
2. Ensure cluster-pool IPAM allocates the missing IPv4 PodCIDR to `CiliumNode/medium-fbfpz-5brrp`.
3. Confirm that the Cilium CNI agent on `medium-fbfpz-5brrp` initializes and pod sandboxes can be created.

Thank you,  
Infrastructure Team
