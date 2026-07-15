# Session 5: How We Operate OCP Clusters

**Day-to-Day Operations, Maintenance, and Cluster Lifecycle**

**Presenter:** Rohit Bhilare — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat
**Duration:** 40 minutes | **Format:** Peer conversation, not a lecture
**Series:** SRE Best Practices — How We Do Things (Session 5 of 14)
**Focus:** Cluster access, lifecycle operations, node management, upgrades, and diagnostics

---

## Agenda

- Cluster Access — How We Get In
- Cluster Lifecycle Operations
- Node Management and Maintenance
- Upgrade Operations — How We Keep Clusters Current
- Diagnostic Data Collection
- Key Takeaways / Best Practices

---

## Cluster Access — How We Get In

### Centralized Access with Least Privilege

All cluster access goes through a **centralized access gateway** — no direct SSH, no shared kubeconfigs, no static credentials.

| Aspect | Our Approach |
|--------|-------------|
| Default access | **Read-only** — SREs can view resources but cannot modify |
| Elevation | Per-command, not per-session — each write action requires explicit elevation |
| Audit trail | Every elevated action is logged with who, what, when, and why |
| Time-limited | Sessions expire automatically after a fixed duration |
| Accountability | You must provide a justification for every elevated action |

### How It Works in Practice

```bash
# Standard read access — works without elevation
oc get nodes
oc get co
oc get pods -A

# Write operations require elevation (per-command)
# Example: draining a node
oc adm drain <node> --ignore-daemonsets --delete-emptydir-data --as cluster-admin
```

> **Principle:** Least privilege by default. Elevate only when needed, only for the specific action. If something goes wrong, we can trace exactly what was done and by whom.

---

## Cluster Lifecycle Operations

### Provisioning Failure Triage

When a cluster fails to provision, our automated investigation layer gathers initial data before an SRE is involved. When manual triage is needed:

**What we check:**

- **Cloud account status** — was the account provisioned correctly?
- **Cluster deployment conditions** — what stage did it fail at?
- **Install logs** — what error did the installer report?
- **Common red herrings** — ingress/auth/console at 96-97% during install is normal (workers not ready yet); skip those

### Deprovisioning and Orphan Cleanup

Stuck uninstalls happen more often than you'd expect. We follow a progression:

1. **Customer self-service first** — let the customer retry the deletion
2. **Disable delete protection** — if the cluster is stuck because of protection flags
3. **Force delete** — last resort, requires justification and tracking ticket

---

## Node Management and Maintenance

### Node Maintenance Process (Aligned with OCP Docs)

When a node needs maintenance (OS update, hardware fix, kernel patching, firmware upgrade), we follow the standard OpenShift node maintenance workflow:

**Step 1 — Mark the node as unschedulable (cordon)**

Prevents the scheduler from placing new pods on this node:

```bash
oc adm cordon <node>

# Verify — STATUS should show SchedulingDisabled
oc get node <node>
```

**Step 2 — Evacuate pods from the node (drain)**

Gracefully moves all pods to other available nodes:

```bash
oc adm drain <node> --ignore-daemonsets --delete-emptydir-data --force
```

| Flag | Why |
|------|-----|
| `--ignore-daemonsets` | DaemonSet pods can't be rescheduled elsewhere — skip them |
| `--delete-emptydir-data` | Pods using emptyDir volumes will lose that data — acknowledge it |
| `--force` | Evict pods not managed by a ReplicaSet/Deployment (standalone pods) |

> If pods have **PodDisruptionBudgets (PDBs)**, the drain respects them — it won't evict more pods than the PDB allows. This protects application availability during maintenance.

**Step 3 — Perform maintenance**

Node is now empty (except DaemonSets). Perform the required work — OS patching, hardware replacement, kernel update, etc.

**Step 4 — Mark the node as schedulable again (uncordon)**

```bash
oc adm uncordon <node>

# Verify — SchedulingDisabled should be gone
oc get node <node>
```

**Step 5 — Verify node health**

```bash
# Node should be Ready
oc get node <node>

# Check that pods are scheduling back
oc get pods --all-namespaces --field-selector spec.nodeName=<node>

# Resource utilization
oc adm top node <node>
```

### Node Replacement (Preferred Over Drain/Reboot)

For managed clusters, replacing the machine is often cleaner than fixing a sick node:

```bash
# Find node → machine mapping
oc get nodes --output="custom-columns=NODE:.metadata.name,\
  MACHINE:.metadata.annotations.machine\.openshift\.io/machine"

# Delete the machine — a new one provisions automatically
oc delete machine -n openshift-machine-api <machine-name>
```

Fresh node, fresh OS, clean state — this is our default approach unless there's a specific reason to preserve the node.

### Infrastructure Node Resizing

When infra nodes are under resource pressure (typically from the monitoring stack or ingress):

```bash
# Check current utilization
oc adm top node -l node-role.kubernetes.io/infra

# Prometheus queries for deeper analysis
# CPU by pod on infra node:
sort_desc(sum by (pod, namespace) (rate(container_cpu_usage_seconds_total{node="<infra_node>"}[5m])))

# Memory by pod on infra node:
sort_desc(sum by (pod, namespace) (container_memory_usage_bytes{node="<infra_node>"}))
```

We use automated tooling to resize infra nodes to larger instance types when utilization exceeds thresholds.

---

## Upgrade Operations — How We Keep Clusters Current

### Upgrade Stages We Monitor

Upgrades run through these stages automatically. We monitor each one:

| Stage | What SRE Watches |
|-------|-----------------|
| **Pre-health check** | No critical alerts firing before upgrade starts |
| **Control plane upgrade** | Control plane nodes upgraded successfully |
| **Worker node upgrade** | Workers draining and upgrading (longest stage) |
| **Post-health check** | DaemonSets, ReplicaSets healthy after upgrade |
| **Completion notification** | Upgrade status synced to management layer |

```bash
# Monitor upgrade progress
oc get clusterversion

# Watch node upgrade progress
oc get nodes -o wide
oc get mcp
```

### When Upgrades Get Blocked

Pre-health check failures are the most common blocker — usually another critical alert is firing:

```bash
# Find degraded operators blocking the upgrade
oc get co | awk 'NR>1 { if($3=="False" || $5=="True") print }'
```

**Resolution:** Fix the degraded operator first. The upgrade auto-proceeds once pre-health checks pass.

---

## Diagnostic Data Collection

### must-gather (Standard Diagnostics)

```bash
TMP_DIR=$(mktemp -d)

# General must-gather — collects everything
oc adm must-gather --dest-dir=$TMP_DIR

# Package for sharing
tar -cJf must-gather.$(date -u +%Y%m%dT%H%M%SZ).tar.xz -C $TMP_DIR ./
```

### Targeted Collection

```bash
# Namespace-specific inspect (faster, smaller output)
oc adm inspect --dest-dir=$TMP_DIR ns/openshift-marketplace

# Collect data for a specific operator (e.g., OpenShift Virtualization)
oc adm must-gather --dest-dir=$TMP_DIR \
  --image=registry.redhat.io/container-native-virtualization/cnv-must-gather-rhel9:v4.17.0
```

### Quick Node Diagnostics

```bash
# Debug a node directly
oc debug node/<node>

# Node logs by service
oc adm node-logs node/<node>
oc adm node-logs -u crio node/<node>
oc adm node-logs -u kubelet node/<node>
```

### When to Collect

- Active incidents where the root cause is unclear
- Before escalating to engineering or vendor support
- For RCA evidence after an outage

> **Important:** must-gather output can contain sensitive data. Always sanitize before uploading to external systems.

---

## Key Takeaways / Best Practices

1. **Use centralized access with least-privilege elevation** — no shared kubeconfigs, no persistent admin sessions, full audit trail
2. **Have runbooks for cluster lifecycle events** — provisioning failures and stuck uninstalls need documented procedures
3. **Follow the cordon → drain → maintain → uncordon workflow** — standard OCP process, no shortcuts
4. **Set PodDisruptionBudgets for critical workloads** — drain respects PDBs and protects application availability
5. **Monitor upgrades stage by stage** — know what stage is running and what to watch for at each step
6. **Collect must-gather during incidents** — if you skip it, you lose the diagnostic snapshot
7. **Replace nodes, don't fix them** — fresh machine, fresh OS, clean state

**Up Next:** Session 6 — Inside SRE: Ask & Discuss Anything
