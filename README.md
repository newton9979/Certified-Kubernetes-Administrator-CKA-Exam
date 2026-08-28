# Certified Kubernetes Administrator (CKA) – 15-Day Intensive Exam Plan

## 🎯 Goal

Prepare for the **Certified Kubernetes Administrator (CKA)** exam in **15 days** with a strong focus on **hands-on practice, troubleshooting, and real-world Kubernetes scenarios**.

> **Strategy:** 70% Hands-on Practice + 30% Theory

The CKA curriculum is heavily weighted toward:

- Troubleshooting — **30%**
- Cluster Architecture, Installation & Configuration — **25%**
- Servicing & Networking — **20%**
- Workloads & Scheduling — **15%**
- Storage — **10%**

The goal is not just to learn Kubernetes commands. The goal is:

> **"Give me a broken Kubernetes cluster and I can find and fix the problem."**

---

# 📚 CKA Curriculum

## 10% — Storage

- Implement StorageClasses and dynamic volume provisioning
- Configure volume types
- Understand access modes
- Understand reclaim policies
- Manage PersistentVolumes
- Manage PersistentVolumeClaims

## 30% — Troubleshooting

- Troubleshoot clusters and nodes
- Troubleshoot cluster components
- Monitor cluster and application resource usage
- Manage and evaluate container output streams
- Troubleshoot Services and networking

## 15% — Workloads & Scheduling

- Understand application Deployments
- Perform rolling updates
- Perform rollbacks
- Use ConfigMaps and Secrets
- Configure workload autoscaling
- Understand robust, self-healing application deployments
- Configure Pod admission and scheduling
- Resource limits
- Node affinity
- Scheduling constraints

## 25% — Cluster Architecture, Installation & Configuration

- Manage RBAC
- Prepare infrastructure for Kubernetes
- Create and manage clusters using kubeadm
- Manage Kubernetes cluster lifecycle
- Configure highly available control planes
- Use Helm and Kustomize
- Understand CNI, CSI, CRI
- Understand CRDs
- Install and configure operators

## 20% — Servicing & Networking

- Understand Pod connectivity
- Configure NetworkPolicies
- ClusterIP
- NodePort
- LoadBalancer
- Endpoints
- Gateway API
- Ingress Controllers
- Ingress resources
- CoreDNS

---

# 🚀 15-Day CKA Intensive Plan

## Daily Study Target

Aim for **7–9 hours per day**.

| Session | Time | Activity |
|---|---:|---|
| Morning | 2 hrs | Learn / Revise Concepts |
| Midday | 3 hrs | Hands-on Kubernetes Labs |
| Evening | 2 hrs | Troubleshooting Scenarios |
| Night | 1–2 hrs | Timed CKA Tasks + Revision |

> **Rule:** Every topic you learn → immediately practice it using `kubectl`.

---

# DAY 1 — Kubernetes Fundamentals & Cluster Architecture

## Theory

- Kubernetes architecture
- Control Plane
- API Server
- etcd
- Scheduler
- Controller Manager
- kubelet
- kube-proxy
- Container Runtime
- Pods
- Namespaces
- Labels and Selectors
- YAML basics

## Hands-on

Practice:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get ns
kubectl get pods -o wide
kubectl describe node <node>
kubectl explain pod
```

Create and manage:

- Pod
- Namespace
- Labels
- Multi-container Pod
- Init Container

## Troubleshooting Practice

Intentionally break Pods:

- Wrong image
- Wrong command
- Wrong port
- Missing ConfigMap
- Missing Secret

### Target

**15–20 tasks**

---

# DAY 2 — Pods, Deployments & ReplicaSets

## Learn

- Pod lifecycle
- ReplicaSet
- Deployment
- Rolling updates
- Rollbacks
- Scaling
- Deployment strategies
- Self-healing

## Practice

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment/nginx nginx=nginx:1.27
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
```

## Practice Target

- 10 Deployment tasks
- 10 troubleshooting tasks

**Total: 20 tasks**

---

# DAY 3 — Scheduling & Workloads

## Learn

- Resource Requests
- Resource Limits
- ResourceQuota
- LimitRange
- NodeSelector
- Node Affinity
- Pod Affinity
- Pod Anti-Affinity
- Taints
- Tolerations
- DaemonSet
- Job
- CronJob

## Hands-on Progression

```text
NodeSelector
     ↓
Node Affinity
     ↓
Taint
     ↓
Toleration
     ↓
DaemonSet
     ↓
Job
     ↓
CronJob
```

## Critical Troubleshooting

You should be able to investigate a Pod stuck in:

```text
Pending
```

and determine the root cause.

### Target

**20 scheduling tasks**

---

# DAY 4 — Services & Networking

## Learn

- ClusterIP
- NodePort
- LoadBalancer
- Endpoints
- EndpointSlices
- Service selectors
- Pod-to-Pod communication
- Service DNS
- CoreDNS

## Hands-on Architecture

```text
Frontend Pod
      ↓
Frontend Service
      ↓
Backend Service
      ↓
Backend Pods
```

## Commands

```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
kubectl exec -it <pod> -- curl <service>
```

## Troubleshooting Scenarios

1. Service has no endpoints
2. Wrong selector
3. Wrong targetPort
4. Pod is not listening
5. DNS failure
6. Service unreachable

### Target

**20 networking tasks**

---

# DAY 5 — Ingress, Gateway API & NetworkPolicy

## Learn

- Ingress
- Ingress Controller
- IngressClass
- Gateway API
- HTTPRoute
- NetworkPolicy
- Pod communication restrictions

## Lab Architecture

```text
Internet
   ↓
Ingress
   ↓
Frontend Service
   ↓
Frontend Pods
   ↓
Backend Service
   ↓
Backend Pods
```

## NetworkPolicy Practice

Example:

```text
Frontend → Backend = ALLOW
Frontend → Database = DENY
Backend  → Database = ALLOW
```

## Troubleshooting

Practice:

```text
Ingress returns 404
Ingress returns 502
Service works internally but not through Ingress
NetworkPolicy blocks traffic
DNS does not resolve
```

### Target

**20 networking tasks**

---

# DAY 6 — Storage

Storage is worth **10%**, so do not spend too much time here.

## Learn

- Volumes
- emptyDir
- hostPath
- PersistentVolume
- PersistentVolumeClaim
- StorageClass
- Dynamic provisioning
- Access Modes
- Reclaim Policies

## Understand

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
StorageClass
 ↓
Storage Backend
```

## Practice

Create:

- StorageClass
- PV
- PVC
- Pod

## Commands

```bash
kubectl get pv
kubectl get pvc
kubectl describe pvc <name>
kubectl describe pv <name>
kubectl get storageclass
```

## Troubleshooting

Practice:

- PVC Pending
- PV Available but not Bound
- Wrong StorageClass
- Access mode mismatch
- Pod cannot mount volume

### Target

**15 tasks**

---

# DAY 7 — ConfigMaps, Secrets & RBAC

## ConfigMaps & Secrets

Learn:

- ConfigMap
- Secret
- Environment variables
- Volume-mounted configuration
- Secret types

## RBAC Architecture

```text
User / ServiceAccount
        ↓
Role / ClusterRole
        ↓
RoleBinding / ClusterRoleBinding
        ↓
Permissions
```

## Practice

Create:

- ServiceAccount
- Role
- RoleBinding
- ClusterRole
- ClusterRoleBinding

## Important Commands

```bash
kubectl auth can-i get pods
kubectl auth can-i create deployments
kubectl auth can-i delete pods
```

### Target

**20 RBAC + configuration tasks**

---

# DAY 8 — kubeadm & Cluster Administration

## Learn

- kubeadm
- kubelet
- Control Plane
- Worker Node
- Bootstrap process
- Certificates
- Cluster lifecycle

## Important Commands

```bash
kubeadm init
kubeadm join
kubeadm token create
kubeadm token list
kubeadm reset
```

## Understand

```text
Linux Server
     ↓
Container Runtime
     ↓
kubeadm
     ↓
Control Plane
     ↓
Worker Node
     ↓
kubelet
```

---

# DAY 9 — Cluster Troubleshooting 🔥

This is one of the most important days because **Troubleshooting carries 30% of the CKA exam**.

## Scenario 1 — Node NotReady

```bash
kubectl get nodes
kubectl describe node <node>
systemctl status kubelet
journalctl -u kubelet
```

## Scenario 2 — API Server Problem

```bash
kubectl get pods -n kube-system
kubectl logs <pod> -n kube-system
```

## Scenario 3 — CoreDNS Problem

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system <coredns>
```

## Scenario 4 — kubelet Problem

```bash
systemctl status kubelet
journalctl -u kubelet
```

### Target

🔥 **25 troubleshooting scenarios**

---

# DAY 10 — Application Troubleshooting 🔥

Focus heavily on the **30% Troubleshooting** section.

## Practice These Problems

```text
Pod CrashLoopBackOff
Pod ImagePullBackOff
Pod Pending
Pod OOMKilled
Pod ContainerCreating
Pod Error
Node NotReady
Service unreachable
DNS failure
Ingress failure
PVC Pending
Application timeout
Wrong configuration
Wrong environment variable
Wrong Secret
Wrong Service selector
Wrong port
```

## Troubleshooting Method

Use this workflow:

```text
kubectl get
      ↓
kubectl describe
      ↓
kubectl logs
      ↓
kubectl exec
      ↓
Check Events
      ↓
Identify Root Cause
      ↓
Fix
      ↓
Verify
```

### Target

🔥 **30 troubleshooting tasks**

---

# DAY 11 — Advanced Workloads & Autoscaling

## Learn

- HPA
- Resource Requests
- Resource Limits
- DaemonSet
- StatefulSet concepts
- Jobs
- CronJobs
- Pod disruption concepts
- Self-healing applications

## HPA Practice

```bash
kubectl autoscale deployment nginx \
  --cpu-percent=50 \
  --min=2 \
  --max=10
```

## Understand

```text
Deployment
    ↓
HPA
    ↓
Resource Requests
    ↓
Metrics
    ↓
Pod Scaling
```

### Target

**20 tasks**

---

# DAY 12 — Helm, Kustomize & CRDs

## Helm

Learn:

```bash
helm repo add
helm search repo
helm install
helm upgrade
helm rollback
helm uninstall
```

Understand:

```text
Chart
 ├── Chart.yaml
 ├── values.yaml
 └── templates/
```

## Kustomize

Practice:

```bash
kubectl apply -k .
kubectl kustomize .
```

## CRD & Operators

Understand:

```text
CRD
 ↓
Custom Resource
 ↓
Operator
```

> Do not spend excessive time here. The objective is CKA operational understanding, not becoming a Kubernetes operator developer.

---

# DAY 13 — Full CKA Mock Exam #1 🔥

Now stop learning new topics.

## 2-Hour Mock Exam

Create **17–20 practical tasks**.

Example:

```text
Task 1  - Fix a broken Deployment
Task 2  - Create a Service
Task 3  - Configure NetworkPolicy
Task 4  - Fix DNS
Task 5  - Create PVC
Task 6  - Configure RBAC
Task 7  - Taint/Toleration
Task 8  - Node Affinity
Task 9  - Rolling update
Task 10 - Rollback
Task 11 - Fix CrashLoopBackOff
Task 12 - Fix Node NotReady
Task 13 - Configure Ingress
Task 14 - Create CronJob
Task 15 - Configure ConfigMap
Task 16 - Configure Secret
Task 17 - Troubleshoot Service
```

## Track Your Performance

Record:

```text
Question
↓
Time Taken
↓
Mistake
↓
Root Cause
↓
Correct Command
```

Do not only check your score.

---

# DAY 14 — Full CKA Mock Exam #2 🔥🔥

Repeat another **2-hour exam**, but make it harder.

## Target

**20 tasks / 120 minutes**

That gives you approximately:

**6 minutes per task**

## Commands You Must Know Quickly

```bash
kubectl get
kubectl describe
kubectl logs
kubectl exec
kubectl edit
kubectl apply
kubectl delete
kubectl explain
kubectl scale
kubectl rollout
kubectl label
kubectl annotate
kubectl taint
kubectl cordon
kubectl drain
kubectl uncordon
kubectl auth can-i
```

---

# DAY 15 — Final Revision + Exam Simulation 🚀

## Morning

Revise your Kubernetes command sheet.

## Afternoon

Do:

**20 rapid-fire troubleshooting scenarios**

Example:

```text
Pod Pending
   ↓
Find reason
   ↓
Fix
   ↓
Verify
```

## Evening

Take your final:

**2-hour full CKA mock exam**

Rules:

- No notes
- No videos
- No Google
- Use only the allowed exam environment/resources
- Treat it exactly like the real exam

## Night

Only revise mistakes.

> **Do NOT start new topics on Day 15.**

---

# 🔥 15-Day Practice Distribution

| Topic | Practice Tasks |
|---|---:|
| Troubleshooting | **70** |
| Cluster Architecture/Admin | **40** |
| Networking | **45** |
| Workloads/Scheduling | **40** |
| Storage | **20** |
| RBAC/Security | **20** |
| ConfigMaps/Secrets | **15** |
| Helm/Kustomize/CRD | **10** |
| Mock Exams | **3 × 17–20** |
| **Total** | **~310+ tasks** |

---

# 🧠 Daily CKA Routine

Use this routine every day:

| Time | Activity |
|---|---|
| 6:00 – 8:00 AM | Theory + Notes |
| 8:00 – 9:00 AM | Break |
| 9:00 AM – 12:00 PM | 🔥 Hands-on Labs |
| 12:00 – 1:00 PM | Lunch |
| 1:00 – 3:00 PM | 🔥 CKA Scenario Tasks |
| 3:00 – 4:00 PM | Break |
| 4:00 – 6:00 PM | 🔥 Troubleshooting |
| 6:00 – 7:00 PM | Break |
| 7:00 – 9:00 PM | Timed CKA Tasks |
| 9:00 – 10:00 PM | Revision + Commands |

---

# ⚡ Most Important Commands to Master

Do not try to memorize 500 commands. Become extremely fast with the following.

## Investigation

```bash
kubectl get
kubectl describe
kubectl logs
kubectl exec
kubectl get events
```

## Pods

```bash
kubectl run
kubectl create
kubectl delete
kubectl edit
kubectl apply
```

## Deployments

```bash
kubectl create deployment
kubectl scale
kubectl set image
kubectl rollout status
kubectl rollout history
kubectl rollout undo
```

## Nodes

```bash
kubectl get nodes
kubectl describe node
kubectl cordon
kubectl drain
kubectl uncordon
```

## Scheduling

```bash
kubectl taint nodes
kubectl label nodes
```

## Networking

```bash
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
kubectl exec
```

## RBAC

```bash
kubectl auth can-i
kubectl create role
kubectl create rolebinding
kubectl create serviceaccount
```

## Storage

```bash
kubectl get pv
kubectl get pvc
kubectl describe pvc
kubectl get storageclass
```

---

# 🔴 CKA Priority Order

Based on the exam weighting:

```text
                    CKA
                     │
       ┌─────────────┼─────────────┐
       │             │             │
Troubleshooting  Architecture  Networking
     30%             25%           20%
       │             │             │
       └─────────────┼─────────────┘
                     │
               Workloads 15%
                     │
                Storage 10%
```

## Priority 1 🔴 — Troubleshooting

You should be able to troubleshoot without hesitation.

## Priority 2 🔴 — Networking

Focus on:

- Services
- DNS
- NetworkPolicy
- Ingress
- Gateway API

## Priority 3 🔴 — Cluster Administration

Focus on:

- Nodes
- kubelet
- Control Plane
- kubeadm
- Cluster lifecycle

## Priority 4 🟠 — Scheduling & Workloads

Focus on:

- Affinity
- Taints
- Tolerations
- Deployments
- HPA
- Jobs
- CronJobs

## Priority 5 🟡 — Storage

Focus on:

- PV
- PVC
- StorageClass
- Dynamic provisioning
- Access modes
- Reclaim policies

---

# 🎯 Final 15-Day Target

Do not aim for:

> "I watched all CKA videos."

Aim for:

> **"I can solve Kubernetes problems under time pressure."**

You should be comfortable with a scenario such as:

```text
Pod is Pending
       ↓
kubectl get pod
       ↓
kubectl describe pod
       ↓
Events show scheduling problem
       ↓
Check node labels / taints / resources
       ↓
Fix affinity / toleration / resources
       ↓
Pod Running
       ↓
Verify application
```

That is the **CKA mindset**.

---

# ✅ Final Checklist Before Exam

- [ ] Pods
- [ ] Deployments
- [ ] ReplicaSets
- [ ] Rolling Updates
- [ ] Rollbacks
- [ ] Services
- [ ] Endpoints
- [ ] EndpointSlices
- [ ] CoreDNS
- [ ] NetworkPolicy
- [ ] Ingress
- [ ] Gateway API basics
- [ ] ConfigMaps
- [ ] Secrets
- [ ] RBAC
- [ ] ServiceAccounts
- [ ] Resource Requests/Limits
- [ ] HPA
- [ ] NodeSelector
- [ ] Node Affinity
- [ ] Pod Affinity
- [ ] Pod Anti-Affinity
- [ ] Taints
- [ ] Tolerations
- [ ] DaemonSets
- [ ] Jobs
- [ ] CronJobs
- [ ] PV
- [ ] PVC
- [ ] StorageClass
- [ ] Dynamic Provisioning
- [ ] kubeadm
- [ ] kubelet
- [ ] Control Plane troubleshooting
- [ ] Node troubleshooting
- [ ] Helm
- [ ] Kustomize
- [ ] CRDs
- [ ] Operators basics
- [ ] 3 full mock exams
- [ ] 300+ hands-on tasks

---

# 🏆 Success Formula

```text
15 Days
   ↓
7–9 Hours/Day
   ↓
70% Hands-on
   ↓
300+ Practical Tasks
   ↓
70+ Troubleshooting Scenarios
   ↓
3 Full Mock Exams
   ↓
Review Every Mistake
   ↓
CKA Exam Ready 🚀
```

## 🔥 Golden Rule

> **Learn → Practice → Break → Troubleshoot → Fix → Repeat**

**Slow is smooth, smooth is fast.**
