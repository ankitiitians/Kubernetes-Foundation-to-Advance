# Week 2: etcd Basics & Backup - Complete Study Notes

> **Course:** Kubernetes Administration  
> **Topic:** etcd Architecture, Backup & Restore  
> **Duration:** 2 Hours  
> **Level:** Intermediate to Advanced

---

## Table of Contents

1. [Introduction - The Real-World Context](#1-introduction---the-real-world-context)
2. [What is etcd?](#2-what-is-etcd)
3. [etcd Architecture & Role in Kubernetes](#3-etcd-architecture--role-in-kubernetes)
4. [Understanding Key-Value Store](#4-understanding-key-value-store)
5. [What Data Does etcd Store?](#5-what-data-does-etcd-store)
6. [etcd Security & Certificates](#6-etcd-security--certificates)
7. [Backup Strategy](#7-backup-strategy)
8. [Restore Procedures](#8-restore-procedures)
9. [Hands-on Labs](#9-hands-on-labs)
10. [Best Practices & Production Tips](#10-best-practices--production-tips)
11. [Troubleshooting Guide](#11-troubleshooting-guide)
12. [Interview Questions](#12-interview-questions)

---

## 1. Introduction - The Real-World Context

### The Friday Evening Disaster Story

**Scenario:** Production fintech cluster failure  
**Time:** Friday, 6:30 PM  
**Company:** Large payment processing company

#### Timeline of Events

| Time | Event |
|------|-------|
| 6:30 PM | Production cluster becomes unresponsive |
| 6:35 PM | API server completely down |
| 6:40 PM | All `kubectl` commands failing |
| 7:00 PM | CEO starts calling the team |
| 11:00 PM | Root cause identified: etcd disk full leading to data corruption |
| 11:08 PM | Backup restored - System back online |

#### Impact
- **Financial:** Rs. 50 lakh transaction processing stopped
- **Users:** 10,000+ customers affected
- **Downtime:** 4.5 hours total

#### Actions Taken (That Did Not Work)
1. Restarted API Server - Failed
2. Checked network connectivity - All connections were fine
3. Reviewed application logs - No clear indication of the issue

#### The Solution
**etcd backup restore - 8 minutes to recovery**

### Key Lesson

**Without backup:** 6-8 hours to rebuild cluster plus complete data loss  
**With backup:** 8 minutes to full recovery with minimal data loss

This demonstrates why **etcd backup is mandatory** for production clusters, not optional.

---

## 2. What is etcd?

### Definition

**etcd** is a **distributed**, **reliable**, **key-value store** that serves as Kubernetes' primary datastore for all cluster data.

Let's break down this definition:

```
etcd = Distributed + Key-Value + Store
         |            |           |
         |            |           └─> Database
         |            └─────────────> Data Model
         └──────────────────────────> Architecture
```

### Understanding "Distributed"

**Distributed** means etcd runs on multiple nodes simultaneously, with data replicated across all nodes.

```
        ┌─────────────┐
        │   etcd-1    │
        │  (Leader)   │
        └──────┬──────┘
               │
        ┌──────┴──────┐
        |             |
   ┌────▼────┐   ┌───▼─────┐
   │ etcd-2  │   │ etcd-3  │
   │(Follower)│  │(Follower)│
   └─────────┘   └─────────┘
```

**Why distributed?**
- **High Availability:** If one node fails, others continue working
- **Data Redundancy:** No single point of failure
- **Fault Tolerance:** Cluster survives partial failures

**Real-world analogy:** Similar to keeping backup copies of important documents in multiple safe locations.

### Understanding "Key-Value"

A **key-value store** works like a dictionary or phone book:

```
Key (Unique Name)           →    Value (Data)
─────────────────────────────    ────────────────────────
"Mom"                       →    9876543210
"Pizza Hut"                 →    1800-208-0123
"Emergency"                 →    112
```

In Kubernetes context:

```
Key                                      →    Value
──────────────────────────────────────────    ────────────────────────────
/registry/pods/default/nginx             →    {pod configuration JSON}
/registry/services/default/kubernetes    →    {service configuration}
/registry/secrets/default/db-password    →    {secret data}
```

**Why key-value model?**
- **Fast lookups:** O(log n) complexity
- **Simple:** No complex queries needed
- **Scalable:** Easy to distribute across nodes
- **Consistent:** Easy to maintain data integrity

### Understanding "Store"

etcd is a **specialized database** designed for:
- Configuration management
- Service discovery
- Distributed coordination
- Metadata storage

**What makes it special?**
- Written in **Go** (fast and efficient)
- Uses **Raft consensus algorithm** (ensures data consistency)
- Supports **watch operations** (real-time change notifications)
- **ACID compliant** (reliable transactions)

---

## 3. etcd Architecture & Role in Kubernetes

### Kubernetes Control Plane Architecture

```
╔═══════════════════════════════════════════════════╗
║         KUBERNETES CONTROL PLANE                  ║
║                                                   ║
║    ┌──────────────┐           ┌──────────────┐  ║
║    │              │   Read/   │              │  ║
║    │  API Server  │◄─────────►│    etcd      │  ║
║    │  (Stateless) │   Write   │ (Stateful)   │  ║
║    │              │           │              │  ║
║    └──────┬───────┘           └──────────────┘  ║
║           │                                      ║
║           │ Watch/Get                            ║
║           │                                      ║
║    ┌──────▼────────────────────────────┐        ║
║    │  Controllers, Scheduler, etc.     │        ║
║    └───────────────────────────────────┘        ║
╚═══════════════════════════════════════════════════╝
```

### Key Relationships

#### API Server and etcd

**API Server:**
- **Stateless component** (no memory of past operations after restart)
- Acts as the frontend to the cluster
- **Only component** that directly communicates with etcd
- Validates and processes all REST operations

**etcd:**
- **Stateful component** (persistent storage)
- Single source of truth for cluster state
- Stores desired and current state of all objects
- Never directly accessed by other components (except API Server)

### Complete Flow: Creating a Deployment

Let's trace what happens when you run:

```bash
kubectl create deployment nginx --image=nginx
```

#### Step-by-Step Breakdown

```
┌─────────────────────────────────────────────────────────┐
│ Step 1: kubectl to API Server                          │
└─────────────────────────────────────────────────────────┘

kubectl sends HTTP POST request:
POST /apis/apps/v1/namespaces/default/deployments
Body: {deployment configuration}

┌─────────────────────────────────────────────────────────┐
│ Step 2: API Server Validation                          │
└─────────────────────────────────────────────────────────┘

API Server performs 3 checks:
1. Authentication: Is this a valid user?
2. Authorization: Does user have permission?
3. Admission: Is the request valid?

┌─────────────────────────────────────────────────────────┐
│ Step 3: Write to etcd (CRITICAL STEP)                  │
└─────────────────────────────────────────────────────────┘

API Server writes to etcd:
Key: /registry/deployments/default/nginx
Value: {
  "apiVersion": "apps/v1",
  "kind": "Deployment",
  "metadata": {"name": "nginx"},
  "spec": {"replicas": 1, ...}
}

IMPORTANT NOTE: No pod is created yet. Only desired state is written.

┌─────────────────────────────────────────────────────────┐
│ Step 4: Deployment Controller Watches etcd             │
└─────────────────────────────────────────────────────────┘

Deployment Controller detects new deployment
→ Creates ReplicaSet
→ Writes ReplicaSet to etcd

┌─────────────────────────────────────────────────────────┐
│ Step 5: ReplicaSet Controller                          │
└─────────────────────────────────────────────────────────┘

ReplicaSet Controller: "I need 1 pod"
→ Creates Pod object
→ Writes Pod to etcd (status: Pending)

┌─────────────────────────────────────────────────────────┐
│ Step 6: Scheduler Selects Node                         │
└─────────────────────────────────────────────────────────┘

Scheduler sees pending pod
→ Selects best node (for example, node-1)
→ Updates pod.spec.nodeName in etcd

┌─────────────────────────────────────────────────────────┐
│ Step 7: Kubelet Creates Container                      │
└─────────────────────────────────────────────────────────┘

Kubelet on node-1 watches etcd for pods assigned to its node
→ Detects new pod assignment
→ Instructs container runtime: "Start nginx container"
→ Pod actually starts running
→ Updates pod status in etcd (status: Running)
```

### Key Insight

Every single operation in Kubernetes follows this pattern:
1. Write desired state to **etcd**
2. Controllers **watch** etcd for changes
3. Controllers **reconcile** current state to match desired state
4. Update current state back to **etcd**

**This is why etcd is called the "brain" of Kubernetes.**

---

## 4. Understanding Key-Value Store

### Basic Concept

A key-value store is the simplest database model. Think of it as a giant HashMap or Dictionary.

```python
# Python example - this is exactly how etcd works
cluster_data = {
    "/registry/pods/default/nginx-abc123": {
        "apiVersion": "v1",
        "kind": "Pod",
        "metadata": {"name": "nginx-abc123"},
        "spec": {"containers": [...]},
        "status": {"phase": "Running"}
    },
    "/registry/services/default/web": {
        "apiVersion": "v1",
        "kind": "Service",
        "metadata": {"name": "web"},
        "spec": {"ports": [...]}
    }
}

# Fast lookup - O(1) or O(log n)
pod_data = cluster_data["/registry/pods/default/nginx-abc123"]
```

### Hierarchical Structure in etcd

etcd organizes keys in a **hierarchical structure** similar to a filesystem:

```
/registry/  (root)
│
├─ pods/
│  ├─ default/
│  │  ├─ nginx-abc123
│  │  ├─ redis-xyz789
│  │  └─ postgres-def456
│  │
│  ├─ kube-system/
│  │  ├─ coredns-123456
│  │  ├─ kube-proxy-789012
│  │  └─ etcd-controlplane
│  │
│  └─ production/
│     ├─ web-app-111
│     └─ api-server-222
│
├─ services/
│  ├─ default/
│  │  ├─ kubernetes
│  │  └─ web-service
│  │
│  └─ production/
│     └─ api-service
│
├─ deployments/
│  ├─ default/
│  │  └─ nginx-deployment
│  │
│  └─ production/
│     ├─ web-deployment
│     └─ api-deployment
│
├─ configmaps/
│  └─ default/
│     └─ app-config
│
├─ secrets/
│  └─ default/
│     └─ db-credentials
│
└─ namespaces/
   ├─ default
   ├─ kube-system
   └─ production
```

### Key Naming Pattern

```
/registry/<resource-type>/<namespace>/<resource-name>
    │           │              │            │
    │           │              │            └─ Unique identifier
    │           │              └────────────── Namespace (isolation)
    │           └───────────────────────────── Resource type
    └───────────────────────────────────────── Base path
```

**Examples:**

| Full Key Path | Resource Type | Namespace | Resource Name |
|---------------|---------------|-----------|---------------|
| `/registry/pods/default/nginx-abc123` | pods | default | nginx-abc123 |
| `/registry/services/kube-system/kube-dns` | services | kube-system | kube-dns |
| `/registry/deployments/production/web` | deployments | production | web |

### Why This Structure?

**Benefits:**

1. **Efficient Queries**
   ```bash
   # Get all pods in default namespace
   GET /registry/pods/default/*
   
   # Get specific pod
   GET /registry/pods/default/nginx-abc123
   
   # Get all services
   GET /registry/services/*
   ```

2. **Namespace Isolation**
   - Each namespace has its own subtree
   - Easy to list or delete all resources in a namespace
   - Natural access control boundaries

3. **Resource Type Filtering**
   - Quickly query all resources of a specific type
   - Efficient for controllers that watch specific resources

---

## 5. What Data Does etcd Store?

### Complete List of Stored Data

#### Resources Stored in etcd

| Category | Resources | Example Keys |
|----------|-----------|--------------|
| **Workloads** | Pods, Deployments, ReplicaSets, StatefulSets, DaemonSets, Jobs, CronJobs | `/registry/pods/default/nginx` |
| **Services** | Services, Endpoints, Ingresses | `/registry/services/default/web` |
| **Configuration** | ConfigMaps, Secrets | `/registry/configmaps/default/app-config` |
| **Storage** | PersistentVolumes, PersistentVolumeClaims, StorageClasses | `/registry/persistentvolumes/pv-001` |
| **RBAC** | Roles, RoleBindings, ClusterRoles, ClusterRoleBindings | `/registry/roles/default/developer` |
| **Cluster** | Namespaces, Nodes, ServiceAccounts | `/registry/namespaces/default` |
| **Events** | Events | `/registry/events/default/pod-created` |
| **Custom** | CRDs, Custom Resources | `/registry/customresources/...` |

#### NOT Stored in etcd

| Data Type | Where It's Actually Stored |
|-----------|---------------------------|
| Container Logs | Node filesystem: `/var/log/pods/` |
| Metrics Data | Metrics Server or Prometheus |
| Application Data | Persistent Volumes or Databases |
| Container Images | Container Registry (Docker Hub, etc.) |
| Image Layers | Node filesystem: `/var/lib/containerd/` |

### Live Demo: Exploring etcd Data

#### Step 1: Access etcd Pod

```bash
# SSH to control plane node
ssh controlplane

# Enter etcd pod
kubectl exec -it etcd-controlplane -n kube-system -- sh

# Set API version
export ETCDCTL_API=3
```

#### Step 2: List All Keys

```bash
# List all keys (warning: can be thousands of entries)
etcdctl get / --prefix --keys-only \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Output:**
```
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/clusterrolebindings/cluster-admin
/registry/clusterroles/admin
/registry/configmaps/default/my-config
/registry/configmaps/kube-system/coredns
/registry/deployments/default/nginx
/registry/pods/default/nginx-abc123
/registry/pods/kube-system/coredns-xyz789
/registry/secrets/default/my-secret
/registry/services/default/kubernetes
... (thousands more)
```

#### Step 3: View Specific Key Value

```bash
# Get a pod's full data
etcdctl get /registry/pods/default/nginx-abc123 \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**Output:** (Binary + JSON mix - Protocol Buffers format)
```
k8s

v1Pod
nginx-abc123
default
...{complete JSON blob with full pod definition}...
```

### Understanding the Data Format

etcd stores Kubernetes objects in **Protocol Buffers** format (efficient binary encoding), but the content is essentially the complete YAML/JSON definition of each resource.

**What's included for each object:**
- API version and kind
- Metadata (name, namespace, labels, annotations)
- Spec (desired state)
- Status (current state)
- Timestamps (creation, deletion)
- Resource version (for optimistic concurrency)

### Simple Rule to Remember

**Rule:** Anything you can get with `kubectl get` is stored in etcd.

```bash
kubectl get pods          # data from: /registry/pods/...
kubectl get services      # data from: /registry/services/...
kubectl get secrets       # data from: /registry/secrets/...
```

---

## 6. etcd Security & Certificates

### Why Security is Critical

#### Security Risk Demonstration

By default, Kubernetes **Secrets are NOT encrypted** in etcd - they are only base64 encoded.

```bash
# View a secret in etcd
etcdctl get /registry/secrets/default/db-password ...

# Output shows:
username: YWRtaW4=          # base64: "admin"
password: cGFzc3dvcmQxMjM=  # base64: "password123"

# Decode it:
echo "cGFzc3dvcmQxMjM=" | base64 -d
# Output: password123
```

**Implication:** Anyone with etcd access can read ALL cluster secrets.

### Security Layers

etcd security has **three layers**:

```
┌─────────────────────────────────────────┐
│ Layer 1: Network Security              │
│ - Firewall rules                        │
│ - Port restrictions (2379, 2380)        │
│ - VPC isolation                         │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ Layer 2: TLS Authentication             │
│ - Certificate-based authentication      │
│ - Encrypted communication               │
│ - Mutual TLS (mTLS)                     │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ Layer 3: Encryption at Rest             │
│ - Data encrypted on disk                │
│ - KMS integration                       │
│ - Envelope encryption                   │
└─────────────────────────────────────────┘
```

### Understanding TLS Certificates

#### Certificate Files in Kubernetes

```bash
# List etcd certificates
ls -la /etc/kubernetes/pki/etcd/

# Output:
-rw------- 1 root root 1675 Dec 10 10:00 ca.key
-rw-r--r-- 1 root root 1099 Dec 10 10:00 ca.crt
-rw------- 1 root root 1679 Dec 10 10:00 server.key
-rw-r--r-- 1 root root 1188 Dec 10 10:00 server.crt
-rw------- 1 root root 1675 Dec 10 10:00 peer.key
-rw-r--r-- 1 root root 1184 Dec 10 10:00 peer.crt
```

#### Three Types of Certificates

**1. CA Certificate (`ca.crt`)**
```
Role: Certificate Authority - root of trust
Purpose: Verifies that other certificates are genuine
Analogy: Government's passport office seal
```

**2. Server Certificate (`server.crt` + `server.key`)**
```
Role: etcd server's identity
Purpose: Clients can verify "this is the real etcd server"
Analogy: etcd's ID card and signature
```

**3. Peer Certificate (`peer.crt` + `peer.key`)**
```
Role: etcd-to-etcd communication (multi-node clusters)
Purpose: etcd nodes authenticate each other
Analogy: How etcd nodes recognize teammates
```

### Why 3 Certificate Parameters?

Every etcd command needs **three certificate parameters**:

```bash
ETCDCTL_API=3 etcdctl <command> \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \    # Verify server
  --cert=/etc/kubernetes/pki/etcd/server.crt \  # My identity
  --key=/etc/kubernetes/pki/etcd/server.key     # My private key
```

#### Airport Security Analogy

```
--cacert (CA Certificate)
  ↓
  Similar to: Government seal on passport office
  Purpose: "Is this passport office genuine?"

--cert (Client Certificate)
  ↓
  Similar to: Your passport
  Purpose: "This is who I am"

--key (Private Key)
  ↓
  Similar to: Your signature
  Purpose: "Proof that passport belongs to me"
```

You need **all three** to pass security.

### Finding Certificate Paths

```bash
# Method 1: Check etcd static pod manifest
cat /etc/kubernetes/manifests/etcd.yaml | grep -E 'cert|key|ca'

# Method 2: Common locations (kubeadm clusters)
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key
```

### Testing Without Certificates

```bash
# Try without certificates - will FAIL
etcdctl get / --endpoints=https://127.0.0.1:2379

# Error:
# context deadline exceeded
# connection refused
```

This proves security is working correctly.

---

## 7. Backup Strategy

### Why Backup is Mission-Critical

#### Real-World Failure Scenarios

**Scenario 1: Accidental Deletion**
```bash
# Developer accidentally runs:
kubectl delete namespace production

# Result: All production resources deleted
# Recovery time without backup: 6-8 hours + data loss
# Recovery time with backup: 8-15 minutes
```

**Scenario 2: Hardware Failure**
- etcd disk corruption
- Multiple node failures (beyond quorum)
- Storage system failure

**Scenario 3: Human Error**
- Wrong kubectl delete command
- Misconfigured RBAC (locked out administrators)
- Failed cluster upgrade

**Scenario 4: Security Incident**
- Ransomware attack
- Malicious insider
- Compromised credentials

### Backup Methods Comparison

| Method | Pros | Cons | Use Case |
|--------|------|------|----------|
| **Built-in Snapshot** | Official method<br>Consistent data<br>Fast execution<br>Small file size | Needs etcd access<br>Requires certificates | **Production (Recommended)** |
| **Volume Snapshot** | Works if etcd is down<br>Storage-level backup | May be inconsistent<br>Larger file size<br>Needs LVM/storage support | Disaster scenarios |
| **Data Directory Copy** | Simple to understand | Requires downtime<br>Large file size<br>Not recommended for production | Development only |

### Backup Strategy - Production Standards

#### Backup Frequency Guidelines

| Cluster Type | Backup Frequency | Retention | RTO | RPO |
|--------------|------------------|-----------|-----|-----|
| Development | Daily | 7 days | 2 hours | 24 hours |
| Staging | Every 6 hours | 14 days | 1 hour | 6 hours |
| Production | Every 1 hour | 30 days | 15 mins | 1 hour |
| Critical Production | Every 15 mins | 90 days | 5 mins | 15 mins |

**Terms:**
- **RTO (Recovery Time Objective):** How quickly you can restore service
- **RPO (Recovery Point Objective):** How much data loss is acceptable

#### Backup Storage Locations (3-2-1 Rule)

```
Primary Backup
  ↓
  Same datacenter, different storage system
  
Secondary Backup
  ↓
  Different datacenter (geo-redundancy)
  
Tertiary Backup
  ↓
  Object storage (S3, GCS, Azure Blob)
```

### Taking a Backup - Step by Step

#### Command Structure

```bash
ETCDCTL_API=3 etcdctl snapshot save <backup-file> \
  --endpoints=<etcd-endpoint> \
  --cacert=<ca-certificate> \
  --cert=<client-certificate> \
  --key=<client-key>
```

#### Complete Example

```bash
# Step 1: Set API version
export ETCDCTL_API=3

# Step 2: Take snapshot with timestamp
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-backup-$(date +%Y%m%d-%H%M%S).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Expected output:
# {"level":"info","ts":"...","msg":"created temporary db file"}
# {"level":"info","ts":"...","msg":"fetching snapshot"}
# {"level":"info","ts":"...","msg":"fetched snapshot","size":"8.5 MB"}
# Snapshot saved at /backup/etcd-backup-20241210-143022.db
```

#### Step 3: Verify Backup Integrity

```bash
# Check backup file
ls -lh /backup/etcd-backup-*.db
# Output: -rw------- 1 root root 8.5M Dec 10 14:30 etcd-backup-20241210-143022.db

# Verify snapshot status
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-backup-20241210-143022.db --write-out=table

# Output:
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 8e6d1a2b |    12458 |       2847 |     8.5 MB |
+----------+----------+------------+------------+
```

**Understanding the output:**
- **HASH:** Checksum for integrity verification
- **REVISION:** etcd revision at backup time (important for restore verification)
- **TOTAL KEYS:** Number of keys stored (typical range: 2000-5000)
- **TOTAL SIZE:** Actual data size

#### Step 4: Secure the Backup

```bash
# Check file permissions (should be 600 - only root can read/write)
ls -l /backup/etcd-backup-*.db
# -rw------- 1 root root  # Correct permissions

# Copy to secure location
scp /backup/etcd-backup-*.db user@backup-server:/backups/k8s-prod/

# Or upload to S3
aws s3 cp /backup/etcd-backup-*.db s3://company-k8s-backups/prod/$(date +%Y%m%d)/

# Optional: Encrypt before storing
gpg --encrypt --recipient admin@company.com etcd-backup-*.db
```

### Automated Backup with CronJob

#### Complete CronJob Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: etcd-backup-script
  namespace: kube-system
data:
  backup.sh: |
    #!/bin/sh
    set -e
    
    # Configuration
    BACKUP_DIR="/backup"
    RETENTION_DAYS=30
    DATE=$(date +%Y%m%d-%H%M%S)
    BACKUP_FILE="${BACKUP_DIR}/etcd-backup-${DATE}.db"
    
    # Create backup directory if it doesn't exist
    mkdir -p ${BACKUP_DIR}
    
    # Take snapshot
    echo "Taking etcd backup: ${BACKUP_FILE}"
    ETCDCTL_API=3 etcdctl snapshot save ${BACKUP_FILE} \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key
    
    # Verify snapshot
    echo "Verifying backup integrity..."
    ETCDCTL_API=3 etcdctl snapshot status ${BACKUP_FILE} --write-out=json > ${BACKUP_FILE}.json
    
    # Check file size (should be > 1MB for non-empty cluster)
    SIZE=$(stat -c%s "${BACKUP_FILE}" 2>/dev/null)
    if [ $SIZE -lt 1048576 ]; then
        echo "ERROR: Backup file too small (${SIZE} bytes)"
        exit 1
    fi
    
    # Cleanup old backups (keep last 30 days)
    echo "Cleaning up old backups..."
    find ${BACKUP_DIR} -name "etcd-backup-*.db" -mtime +${RETENTION_DAYS} -delete
    
    # Optional: Upload to S3
    # aws s3 cp ${BACKUP_FILE} s3://company-backups/etcd/
    
    echo "Backup completed successfully: ${BACKUP_FILE}"

---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: etcd-backup
  namespace: kube-system
spec:
  schedule: "0 */6 * * *"  # Every 6 hours
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      template:
        spec:
          hostNetwork: true
          nodeName: controlplane  # Run only on control plane node
          restartPolicy: OnFailure
          containers:
          - name: etcd-backup
            image: registry.k8s.io/etcd:3.5.9-0
            command: ["/bin/sh", "-c"]
            args:
            - |
              sh /scripts/backup.sh
            volumeMounts:
            - name: etcd-certs
              mountPath: /etc/kubernetes/pki/etcd
              readOnly:
