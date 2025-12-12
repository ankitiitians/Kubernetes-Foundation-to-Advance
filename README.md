
# **Kubernetes — Foundation to Advance (10-Week Learning Roadmap)**

A complete, structured Kubernetes curriculum progressing from core fundamentals to real-world production expertise. Includes guided labs, deep-dive sessions, practical demos, and interview preparation.

---

# **Table of Contents**

1. [Course Overview](#course-overview)
2. [Week 1: Core Concepts](#week-1-core-concepts-foundation)
3. [Week 2: Advanced Core and Tooling](#week-2-advanced-core-and-tooling)
4. [Week 3: Scheduling and Resource Management](#week-3-scheduling-and-resource-management)
5. [Week 4: Application Lifecycle](#week-4-application-lifecycle)
6. [Week 5: Cluster Maintenance and Upgrades](#week-5-cluster-maintenance-and-upgrades)
7. [Week 6: Security Deep Dive](#week-6-security-deep-dive)
8. [Week 7: Networking and Storage](#week-7-networking-and-storage)
9. [Week 8: Advanced and Real-World Concepts](#week-8-advanced-and-real-world-concepts)
10. [Week 9: Observability and Troubleshooting](#week-9-observability-and-troubleshooting)
11. [Week 10: Interview and Certification Preparation](#week-10-interview-and-certification-preparation)
12. [Tech Stack](#tech-stack)
13. [About This Repository](#about-this-repository)

---

# **Course Overview**

This 10-week program is designed to build a strong Kubernetes foundation and progress toward advanced, production-ready skills. Each week includes hands-on labs aligned with real DevOps, SRE, and cloud engineering practices.

---

# **Week 1: Core Concepts (Foundation)**

### Topics

* Introduction to Kubernetes
* Kubernetes vs Docker Swarm
* Control Plane and Worker Node Architecture
* Pods, ReplicaSets, Deployments
* Service Types: ClusterIP, NodePort, LoadBalancer, ExternalName
* Namespaces, YAML basics, kubectl fundamentals

### Labs

* Deploy your first Pod and Service
* Explore kubectl commands
* Create and switch namespaces

---

# **Week 2: Advanced Core and Tooling**

### Topics

* etcd introduction and backup fundamentals
* API Server overview
* Controller Manager overview
* Kubelet deep dive
* Container runtime fundamentals

### Labs

* Run a busybox pod for debugging
* Take an etcd snapshot

---

# **Week 3: Scheduling and Resource Management**

### Topics

* Labels, Selectors, Annotations
* NodeSelector, Affinity, Anti-Affinity
* Taints and Tolerations
* Resource Requests and Limits
* Pod Priority and Preemption
* DaemonSets, Jobs, CronJobs

### Labs

* Schedule workloads using taints and tolerations
* Configure CPU and memory limits
* Create a CronJob for backups

---

# **Week 4: Application Lifecycle**

### Topics

* ConfigMaps and Secrets
* Multi-container Pod patterns
* Init Containers
* Probes: Liveness, Readiness, Startup
* Rolling Updates and Rollbacks
* StatefulSets
* HPA and VPA basics

### Labs

* Deploy a multi-tier application using ConfigMaps and Secrets
* Test rolling update and rollback behavior
* Enable HPA scaling

---

# **Week 5: Cluster Maintenance and Upgrades**

### Topics

* Cluster provisioning with kubeadm, kind, and minikube
* Kubernetes cluster upgrade process
* etcd restore and compaction deep dive
* Authentication and Authorization (RBAC)
* Admission Controllers
* Certificates and TLS setup

### Labs

* Create RBAC roles and role bindings
* Backup and restore etcd
* Configure an admission controller policy

---

# **Week 6: Security Deep Dive**

### Topics

* TLS in Kubernetes
* Network Policies
* Pod Security Standards, Kyverno, OPA Gatekeeper
* Secrets encryption at rest
* Security Context (runAsUser, capabilities)
* Runtime security: Falco, eBPF
* Image scanning tools: Trivy, cosign

### Labs

* Apply NetworkPolicies to isolate workloads
* Deploy a pod with restricted security context
* Scan container images with Trivy

---

# **Week 7: Networking and Storage**

### Topics

* Kubernetes Networking Model
* kube-proxy (iptables vs IPVS)
* CNI plugins: Calico, Cilium, Flannel
* CoreDNS internals
* Ingress Controllers: NGINX, Traefik, Istio
* PersistentVolumes and PersistentVolumeClaims
* StorageClasses and dynamic provisioning
* CSI drivers, snapshots, volume expansion

### Labs

* Deploy an NGINX Ingress
* Create a PVC and mount it to a Pod
* Test service-to-service communication

---

# **Week 8: Advanced and Real-World Concepts**

### Topics

* Custom Resource Definitions (CRDs) and Operators
* API Aggregation Layer
* GitOps workflows with ArgoCD and FluxCD
* Helm best practices
* CI/CD pipelines: Tekton, Jenkins, GitHub Actions
* Cluster API (CAPI)
* Multi-cluster management with Rancher, ArgoCD, KubeFed

### Labs

* Write and deploy a CRD
* Set up GitOps with ArgoCD
* Deploy a canary release using Argo Rollouts

---

# **Week 9: Observability and Troubleshooting**

### Topics

* Metrics pipelines: metrics-server, kube-state-metrics, Prometheus Operator
* Logging stacks: Fluentd, Fluent Bit, Loki, EFK
* Distributed tracing: Jaeger, OpenTelemetry
* Alerting with Alertmanager
* SLO/SLI design and runbooks
* Troubleshooting pods, services, DNS, and node operations

### Labs

* Install Prometheus and Grafana
* Debug a CrashLoopBackOff pod
* Debug DNS resolution issues

---

# **Week 10: Interview and Certification Preparation**

### Topics

* CKA, CKAD, CKS domain breakdown
* Common Kubernetes interview questions
* Troubleshooting strategies and debugging flows
* Production best practices
* Mock interview session
* Final review and Q&A

### Labs

* Work through hands-on troubleshooting scenarios
* Attempt CKA/CKAD mock exam tasks
* Participate in mock interviews

---

# **Tech Stack**

* Kubernetes
* Docker / Container Runtime
* YAML
* Helm
* GitOps (ArgoCD, FluxCD)
* Prometheus, Grafana
* Logging stacks (EFK, Loki)
* CI/CD (Tekton, Jenkins, GitHub Actions)
* Linux fundamentals

---

# **About This Repository**

This repository serves as a structured learning pathway for anyone preparing for DevOps, SRE, Cloud Engineering roles, or Kubernetes certifications. Each section is designed to build practical skills through hands-on labs and real-world scenarios.

---


