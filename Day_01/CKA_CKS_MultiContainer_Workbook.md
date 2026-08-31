# CKA/CKS Multi-Container Pods & Sidecar Logging Workbook

## Author Notes

This workbook is designed for:

* CKA (Certified Kubernetes Administrator)
* CKS (Certified Kubernetes Security Specialist)

Format:

* Question
* Your Task
* Hints
* Solution
* Explanation
* Troubleshooting

---

# Table of Contents

1. Multi-Container Pods
2. Sidecar Containers
3. Native Sidecars
4. Shared Volumes
5. Same-Pod Networking
6. Security Hardening
7. Linux Capabilities
8. Troubleshooting Labs
9. Combined CKS Scenarios
10. Quick Revision Notes
11. Useful Commands

---

# Section 1: Multi-Container Pods

## Concept

A Pod can contain multiple containers.

Example:

```text
Pod
├── app
└── log-agent
```

Containers share:

* Network namespace
* Volumes
* Lifecycle

Containers do NOT share:

* Process namespace (unless enabled)
* Filesystem (unless mounted)

---

# Question 1 – Sidecar Logging

## Scenario

Create a Pod named:

```text
secure-logger
```

Namespace:

```text
security-lab
```

Requirements:

Application container:

```text
name: app
image: busybox:1.36
```

Write:

```text
application started
```

to:

```text
/var/log/app/app.log
```

every 5 seconds.

Sidecar:

```text
name: log-agent
image: busybox:1.36
```

Must continuously read:

```text
/var/log/app/app.log
```

Use:

```text
tail -F
```

Shared volume:

```text
emptyDir
name: log-volume
```

---

## Your Task

Build the Pod YAML.

---

## Hints

* Use emptyDir
* Mount same volume in both containers
* Use tail -F

---

## Solution

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-logger
  namespace: security-lab

spec:
  containers:

  - name: app
    image: busybox:1.36
    command:
    - sh
    - -c
    - |
      while true; do
        echo "application started" >> /var/log/app/app.log
        sleep 5
      done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  - name: log-agent
    image: busybox:1.36
    command:
    - sh
    - -c
    - tail -F /var/log/app/app.log

    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

---

## Explanation

Both containers mount the same volume.

```text
app
 ↓ writes

emptyDir

 ↓ reads
log-agent
```

---

## Troubleshooting

Check pod:

```bash
kubectl get pod -n security-lab
```

Check sidecar logs:

```bash
kubectl logs secure-logger -n security-lab -c log-agent
```

---

# Section 2: Native Sidecars

## Concept

Native Sidecars run as:

```yaml
initContainers:
```

with:

```yaml
restartPolicy: Always
```

Available in newer Kubernetes releases.

---

# Question 2 – Native Sidecar

## Scenario

Create:

```text
logger
```

Namespace:

```text
cka-lab
```

Requirements:

Sidecar:

```yaml
initContainers:
- name: log-agent
```

Image:

```text
busybox:1.36
```

Command:

```bash
tail -F /var/log/app/app.log
```

Application:

```yaml
containers:
- name: app
```

Write timestamp every 3 seconds.

---

## Verification

```bash
kubectl logs logger -n cka-lab -c log-agent
```

---

## Explanation

Native sidecar starts before application container.

---

# Section 3: Shared Volume Troubleshooting

# Question 3

Given:

```yaml
containers:
- name: app
  volumeMounts:
  - name: log-volume

- name: log-agent
  volumeMounts:
  - name: logs

volumes:
- name: log-volume
```

Problem:

Sidecar cannot see log file.

---

## Your Task

Find and fix issue.

---

## Hint

Compare:

```text
app volume name
sidecar volume name
volume definition
```

---

## Solution

```yaml
containers:
- name: log-agent
  volumeMounts:
  - name: log-volume
```

---

## Explanation

Volume names must match exactly.

---

## Troubleshooting

```bash
kubectl describe pod POD_NAME
```

---

# Section 4: Same-Pod Communication

## Concept

Containers inside a Pod share:

```text
Network Namespace
```

Therefore:

```text
localhost
```

works between containers.

---

# Question 4

Create Pod:

```text
web-test
```

Containers:

```text
web
tester
```

Web runs nginx.

Tester connects using:

```text
localhost:80
```

---

## Answer

Because both containers share the same network namespace.

```text
Pod
├── web
└── tester

Shared Network
```

---

# Section 5: CKS Security

# Question 5

Requirement:

Logging sidecar must NOT modify application files.

---

## Your Task

Mount volume as read-only.

---

## Solution

```yaml
volumeMounts:
- name: log-volume
  mountPath: /var/log/app
  readOnly: true
```

---

## Explanation

Application:

```text
read/write
```

Sidecar:

```text
read-only
```

---

# Section 6: Least Privilege

# Question 6

Security team discovers:

```text
log-agent runs as root
```

Requirement:

* Non-root
* No privilege escalation
* Read-only filesystem

---

## Solution

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000

  allowPrivilegeEscalation: false

  readOnlyRootFilesystem: true
```

---

## Explanation

Applies least-privilege principle.

---

# Section 7: SecurityContext Placement

# Question 7

Requirement:

Only sidecar must run as non-root.

Do not affect application container.

---

## Solution

```yaml
containers:

- name: log-agent
  securityContext:
    runAsNonRoot: true
```

---

## Explanation

Use container-level securityContext.

Avoid Pod-level configuration.

---

# Section 8: Linux Capabilities

# Question 8

Security scan reports:

```text
Unnecessary Linux capabilities
```

---

## Your Task

Drop all capabilities.

---

## Solution

```yaml
securityContext:
  capabilities:
    drop:
    - ALL
```

---

## Explanation

Removes kernel privileges not required.

---

# Section 9: Combined CKS Scenario

# Question 9

Create:

```text
Pod: secure-logger
Namespace: security-lab
```

Containers:

```text
app
log-agent
```

Requirements:

* Shared emptyDir
* Read-only sidecar mount
* runAsNonRoot
* allowPrivilegeEscalation false
* Drop ALL capabilities
* Read-only root filesystem

---

## Security Checklist

```text
☑ Multi-container Pod
☑ Shared emptyDir
☑ Read-only volume
☑ runAsNonRoot
☑ allowPrivilegeEscalation false
☑ capabilities drop ALL
☑ readOnlyRootFilesystem
```

---

# Final Practice Challenge

Create:

```text
Pod: secure-logger
Namespace: cks-lab
```

Requirements:

Application:

```text
Writes logs
```

Sidecar:

```text
tail -F
```

Shared Volume:

```text
emptyDir
name: logs
```

Security:

```text
runAsNonRoot
allowPrivilegeEscalation false
drop ALL capabilities
readOnlyRootFilesystem
```

Architecture:

```text
                secure-logger
                     Pod
                      │
          ┌───────────┴───────────┐
          │                       │
        app                   log-agent
          │                       │
     read/write                read-only
          │                       │
          └───────┬───────────────┘
                  │
               emptyDir
                  │
              app.log
```

---

# CKA Quick Revision

Focus on:

* Pods
* Multi-container Pods
* Sidecars
* Native Sidecars
* Init Containers
* emptyDir
* ConfigMaps
* Secrets
* Networking
* localhost
* kubectl logs

---

# CKS Quick Revision

Focus on:

* SecurityContext
* runAsNonRoot
* runAsUser
* fsGroup
* Capabilities
* Seccomp
* AppArmor
* Read-only root filesystem
* Read-only volumes
* Least Privilege
* Pod Security Standards

---

# Frequently Used Commands

## Logs

```bash
kubectl logs POD
```

Container logs:

```bash
kubectl logs POD -c CONTAINER
```

Follow logs:

```bash
kubectl logs -f POD
```

---

## Describe

```bash
kubectl describe pod POD
```

---

## Exec

```bash
kubectl exec -it POD -- sh
```

Container:

```bash
kubectl exec -it POD -c CONTAINER -- sh
```

---

## YAML Export

```bash
kubectl get pod POD -o yaml
```

---

## Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Dry Run

```bash
kubectl run nginx \
--image=nginx \
--dry-run=client \
-o yaml
```

---

# Exam Tips

1. Always check namespace first.

```bash
kubectl config set-context --current --namespace=<ns>
```

2. Use dry-run whenever possible.

3. Verify:

```bash
kubectl get all
```

4. Check logs before changing YAML.

5. For CKS, always think:

```text
Can I remove privileges?
Can I run as non-root?
Can I make filesystem read-only?
Can I drop capabilities?
```

---

# End of Workbook

Recommended next topics:

* Init Containers
* Ephemeral Containers
* ConfigMaps
* Secrets
* SecurityContext
* Network Policies
* Pod Security Standards
* RuntimeClass
* Seccomp
* AppArmor
* Falco
* Image Scanning
* Supply Chain Security
