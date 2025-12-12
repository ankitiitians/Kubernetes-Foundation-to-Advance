# Kubernetes-Foundation-to-Advance

Week 1: Core Concepts (Foundation)
•	Introduction to Kubernetes
•	Kubernetes vs Docker Swarm
•	Kubernetes Architecture (Control Plane, Worker Nodes)
•	Pods, ReplicaSets, Deployments
•	Services (ClusterIP, NodePort, LoadBalancer, ExternalName)
•	Namespaces, YAML basics, kubectl fundamentals
Labs:
•	Deploy first Pod and Service
•	Explore kubectl commands
•	Create and switch namespaces
 
Week 2: Advanced Core + Tooling
•	etcd introduction & backup basics
•	API server overview
•	Controller Manager overview
•	Kubelet deep dive
•	Container runtime basics
Labs:
•	Run busybox for debugging
•	Take etcd snapshot
 
Week 3: Scheduling & Resource Management
•	Labels, Selectors, Annotations
•	Node Selector, Affinity, Anti-Affinity
•	Taints and Tolerations
•	Resource Requests & Limits
•	Pod Priority & Preemption
•	Daemon Sets, Jobs, CronJobs
Labs:
•	Deploy workload with taints & tolerations
•	Configure CPU/memory limits
•	Create a CronJob for backups
 
Week 4: Application Lifecycle
•	ConfigMaps & Secrets
•	Multi-container Pods (patterns)
•	Init Containers
•	Probes (Liveness, Readiness, Startup)
•	Rolling Updates & Rollbacks
•	Stateful Sets
•	HPA & VPA basics
Labs:
•	Deploy multi-tier app using ConfigMaps & Secrets
•	Test rolling update vs rollback
•	Enable HPA scaling
 
Week 5: Cluster Maintenance & Upgrades
•	Cluster provisioning (kubeadm, kind, minikube)
•	Cluster upgrades
•	etcd deep dive: restore, compaction
•	Authentication & Authorization (RBAC)
•	Admission Controllers
•	Certificates & TLS setup
Labs:
•	Create RBAC roles and bind them
•	Backup and restore etcd
•	Configure admission controller policy
 
Week 6: Security Deep Dive
•	TLS in Kubernetes
•	Network Policies
•	Pod Security Standards, Kyverno, OPA Gatekeeper
•	Secrets encryption at rest
•	Security Context (runAsUser, capabilities)
•	Runtime security: Falco, eBPF
•	Image scanning (Trivy, cosign)
Labs:
•	Apply NetworkPolicy to isolate workloads
•	Deploy pod with restricted security context
•	Scan image with Trivy
 
Week 7: Networking & Storage
•	Kubernetes Networking Model
•	kube-proxy (iptables vs ipvs)
•	CNI plugins: Calico, Cilium, Flannel
•	DNS with CoreDNS
•	Ingress controllers (NGINX, Traefik, Istio)
•	Persistent Volumes & PVCs
•	Storage Classes & dynamic provisioning
•	CSI drivers, snapshots, volume expansion
Labs:
•	Deploy ingress with NGINX
•	Create PVC and bind to pod
•	Test service-to-service communication
 
Week 8: Advanced & Real-World
•	CRDs & Operators
•	API Aggregation Layer
•	GitOps (ArgoCD, FluxCD)
•	Helm best practices
•	CI/CD pipelines (Tekton, Jenkins, GitHub Actions)
•	Cluster API (CAPI)
•	Multi-cluster management (Rancher, ArgoCD, KubeFed)
Labs:
•	Write and deploy CRD
•	Setup GitOps with ArgoCD
•	Deploy canary release with Argo Rollouts
 
Week 9: Observability & Troubleshooting
•	Metrics: metrics-server, kube-state-metrics, Prometheus Operator
•	Logging: Fluentd, Fluent Bit, Loki, EFK
•	Distributed tracing: Jaeger, OpenTelemetry
•	Alerting with Alertmanager
•	Designing SLOs/SLIs, runbooks
•	Debugging pod failures, service/DNS issues, node drain/cordon
Labs:
•	Install Prometheus + Grafana
•	Debug CrashLoopBackOff pod
•	Debug DNS resolution issue
 
Week 10: Interview & Exam Prep
•	CKA/CKAD/CKS domains
•	Common interview questions
•	Debugging flows
•	Best practices & production readiness
•	Mock interview session
•	Final review and Q&A
Labs:
•	Solve troubleshooting labs
•	Attempt CKA/CKAD mock exam tasks
•	Conduct mock interviews in pairs
