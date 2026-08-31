# CKA / CKS Multi-Container Pods & Sidecar Logging — Practice Workbook

## Purpose

This workbook is designed for hands-on CKA and CKS preparation.

Each scenario follows:

**Question → Your Task → Hints → Solution → Explanation → Troubleshooting**

---

# 1. Core Concepts

## Multi-Container Pod

A Pod can contain multiple containers that share:

- Network namespace
- IP address
- localhost
- Volumes
- Pod lifecycle

Typical pattern:

```text
             Pod
              |
       +------+------+
       |             |
      app        log-agent
       |             |
       +------+------+
              |
          shared volume
```

Common multi-container patterns:

- Sidecar
- Adapter
- Ambassador

For the CKA, the sidecar pattern is especially useful for logging and helper processes.

---

# 2. emptyDir Shared Volume

`emptyDir` creates temporary storage for a Pod.

Example:

```yaml
volumes:
- name: log-volume
  emptyDir: {}
```

Two containers can mount the same volume:

```yaml
volumeMounts:
- name: log-volume
  mountPath: /var/log/app
```

Important:

- The volume exists as long as the Pod exists.
- Containers in the same Pod can share it.
- It is commonly used for application logs and sidecar processing.
- Data is lost when the Pod is removed.

---

# 3. Sidecar Logging Pattern

Typical architecture:

```text
app container
    |
    | writes
    v
/var/log/app/app.log
    ^
    |
    | reads
    |
log-agent sidecar
```

The application writes the log and the sidecar continuously reads it.

A common command is:

```bash
tail -F /var/log/app/app.log
```

`-F` is useful because it continues following the file and can handle log-file recreation/rotation better than a simple `tail`.

---

# 4. CKA vs CKS Focus

## CKA

Focus on whether you can:

- Create Pods
- Configure multiple containers
- Configure init containers
- Configure native sidecars
- Configure volumes
- Troubleshoot volume mounts
- Check container-specific logs
- Understand Pod networking
- Use `localhost` between containers in the same Pod

## CKS

Focus on securing the workload:

- `securityContext`
- `runAsNonRoot`
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true`
- Linux capabilities
- Read-only volume mounts
- Least privilege
- Pod Security concepts

---

# Question 1 — Sidecar Logging

## Question

Create a Pod named `secure-logger` in namespace `security-lab`.

Requirements:

- Main container: `app`
- Image: `busybox:1.36`
- Write `application started` to `/var/log/app/app.log` every 5 seconds.
- Sidecar container: `log-agent`
- Image: `busybox:1.36`
- Sidecar continuously reads `/var/log/app/app.log`.
- Both containers share an `emptyDir` volume named `log-volume`.
- Sidecar must use `tail -F`.

## Your Task

Create the namespace and Pod, then verify both containers.

## Hints

Think about:

- `emptyDir`
- `volumeMounts`
- Container commands
- `tail -F`
- `kubectl logs -c`

## Solution

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: security-lab
---
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
    - /bin/sh
    - -c
    - |
      while true; do
        echo "application started $(date)" >> /var/log/app/app.log
        sleep 5
      done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  - name: log-agent
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - tail -F /var/log/app/app.log
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

Apply:

```bash
kubectl apply -f secure-logger.yaml
```

Verify:

```bash
kubectl get pod -n security-lab
kubectl describe pod secure-logger -n security-lab
kubectl logs secure-logger -n security-lab -c app
kubectl logs secure-logger -n security-lab -c log-agent
```

## Explanation

The two containers mount the same `emptyDir` volume at `/var/log/app`.

The application writes:

```text
/var/log/app/app.log
```

The sidecar sees the same file because both containers use:

```yaml
name: log-volume
```

---

# Question 2 — Native Sidecar

## Question

Create a Pod called `logger` in namespace `cka-lab`.

The Pod must contain:

- Native sidecar named `log-agent`
- Image: `busybox:1.36`
- Main container named `app`
- Image: `busybox:1.36`

Requirements:

- Use `initContainers`.
- Set `restartPolicy: Always`.
- The native sidecar runs:

```bash
tail -F /var/log/app/app.log
```

- The main container writes a timestamp every 3 seconds.

## Your Task

Create the Pod and verify the sidecar logs.

## Hints

A native sidecar is declared under:

```yaml
initContainers:
```

and uses:

```yaml
restartPolicy: Always
```

## Solution

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cka-lab
---
apiVersion: v1
kind: Pod
metadata:
  name: logger
  namespace: cka-lab
spec:
  restartPolicy: Always

  initContainers:
  - name: log-agent
    image: busybox:1.36
    restartPolicy: Always
    command:
    - /bin/sh
    - -c
    - tail -F /var/log/app/app.log
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  containers:
  - name: app
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - |
      while true; do
        echo "Application timestamp: $(date)" >> /var/log/app/app.log
        sleep 3
      done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

Verify:

```bash
kubectl get pod logger -n cka-lab
kubectl logs logger -n cka-lab -c log-agent
```

---

# Question 3 — Shared Volume Troubleshooting

## Question

You are given:

```yaml
spec:
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  - name: log-agent
    image: busybox
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

The `log-agent` cannot see the application's log file.

## Your Task

Identify the problem and fix the YAML.

## Hint

Compare:

```text
app volume name       = ?
log-agent volume name = ?
volumes volume name   = ?
```

## Solution

The problem is:

```yaml
log-agent:
  name: logs
```

but the actual volume is:

```yaml
name: log-volume
```

Fix:

```yaml
- name: log-agent
  image: busybox
  volumeMounts:
  - name: log-volume
    mountPath: /var/log/app
```

All three references must match:

```text
app          -> log-volume
log-agent    -> log-volume
volumes      -> log-volume
```

## Troubleshooting Commands

```bash
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
kubectl logs <pod-name> -c app
kubectl logs <pod-name> -c log-agent
```

---

# Question 4 — Same Pod Communication

## Question

Create a Pod called `web-test` with:

- Container `web`
- Container `tester`

The `web` container runs nginx.

The `tester` container periodically connects to:

```text
localhost:80
```

## Your Task

Explain why `tester` can use `localhost:80`.

## Solution

Containers inside the same Pod share the Pod's network namespace.

Architecture:

```text
              Pod
               |
       shared network namespace
               |
        +------+------+
        |             |
       web          tester
        |             |
      :80         localhost:80
```

Therefore, `localhost` refers to the same network namespace.

## Verification

```bash
kubectl exec -it web-test -c tester -- wget -qO- http://localhost:80
```

If the image does not contain `wget`, use another suitable HTTP client/image for testing.

---

# Question 5 — CKS Read-Only Volume

## Question

The logging sidecar must not be able to modify application files.

The application writes:

```text
/var/log/app/app.log
```

## Your Task

Make the sidecar's volume mount read-only.

## Solution

```yaml
volumeMounts:
- name: log-volume
  mountPath: /var/log/app
  readOnly: true
```

## Security Concept

```text
Application
    |
    | read/write
    v
 emptyDir
    ^
    |
    | read-only
    |
Sidecar
```

This follows the principle of least privilege.

---

# Question 6 — CKS Least Privilege

## Question

A Pod contains:

```text
app
log-agent
```

The logging sidecar only needs to read a log file.

A security scan reports that the sidecar is running as root.

## Your Task

Modify `log-agent` so it:

- Does not run as root.
- Uses a non-root user.
- Disables privilege escalation.
- Uses a read-only root filesystem.

## Solution

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

Example:

```yaml
- name: log-agent
  image: busybox:1.36
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
```

## Important

A read-only root filesystem means the container cannot write to its normal root filesystem.

If the application requires writable temporary storage, explicitly mount an appropriate volume such as `emptyDir`.

---

# Question 7 — Container-Level SecurityContext

## Question

You have:

```yaml
containers:
- name: app
  image: nginx

- name: log-agent
  image: busybox
```

Requirement:

Only `log-agent` must run as non-root.

Do not change the security settings of `app`.

## Your Task

Choose the correct location for `securityContext`.

## Solution

Use a container-level `securityContext`:

```yaml
containers:
- name: app
  image: nginx

- name: log-agent
  image: busybox
  securityContext:
    runAsNonRoot: true
```

## Explanation

If the security requirement applies to one container, configure that container.

---

# Question 8 — Linux Capabilities

## Question

A security scan reports that `log-agent` has unnecessary Linux capabilities.

Requirement:

Drop all Linux capabilities from the sidecar.

## Solution

```yaml
securityContext:
  capabilities:
    drop:
    - ALL
```

Example:

```yaml
- name: log-agent
  image: busybox:1.36
  securityContext:
    capabilities:
      drop:
      - ALL
```

## CKS Memory

```text
Unnecessary privileges
        |
        v
Drop capabilities
        |
        v
capabilities:
  drop:
  - ALL
```

---

# Question 9 — Combined CKS Scenario

## Question

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

Images:

```text
app       -> nginx
log-agent -> busybox
```

Requirements:

- Both containers share an `emptyDir`.
- Volume name must be `logs`.
- Application logs are stored at `/var/log/app/app.log`.
- Sidecar reads the application log.
- Sidecar volume mount must be read-only.
- Sidecar runs as non-root.
- Privilege escalation is disabled.
- All Linux capabilities are dropped.
- Root filesystem is read-only.

## Your Task

Create the complete Pod.

## Solution

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: security-lab
---
apiVersion: v1
kind: Pod
metadata:
  name: secure-logger
  namespace: security-lab
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  - name: log-agent
    image: busybox:1.36
    command:
    - /bin/sh
    - -c
    - tail -F /var/log/app/app.log
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: logs
      mountPath: /var/log/app
      readOnly: true

  volumes:
  - name: logs
    emptyDir: {}
```

## Security Checklist

```text
[ ] Multi-container Pod
[ ] Shared emptyDir
[ ] Correct volume name
[ ] Sidecar reads application logs
[ ] Sidecar volume is read-only
[ ] runAsNonRoot
[ ] allowPrivilegeEscalation: false
[ ] capabilities.drop: ALL
[ ] readOnlyRootFilesystem: true
```

---

# Question 10 — Troubleshoot a Sidecar

## Scenario

The Pod is running:

```text
secure-logger   2/2 Running
```

But:

```bash
kubectl logs secure-logger -c log-agent
```

shows no application logs.

## Your Task

Troubleshoot step by step.

## Solution Workflow

### Step 1 — Check both containers

```bash
kubectl get pod secure-logger -o wide
```

### Step 2 — Check application logs

```bash
kubectl logs secure-logger -c app
```

### Step 3 — Check sidecar logs

```bash
kubectl logs secure-logger -c log-agent
```

### Step 4 — Check mounts

```bash
kubectl describe pod secure-logger
```

Look for:

```text
Mounts:
Volumes:
```

### Step 5 — Verify the file from the application container

```bash
kubectl exec secure-logger -c app -- ls -l /var/log/app
```

### Step 6 — Verify from the sidecar

```bash
kubectl exec secure-logger -c log-agent -- ls -l /var/log/app
```

### Step 7 — Check volume names

The same volume must be referenced:

```yaml
volumeMounts:
- name: logs
```

and:

```yaml
volumes:
- name: logs
```

---

# Question 11 — Read-Only Root Filesystem Troubleshooting

## Scenario

You configured:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

The sidecar crashes because it tries to write temporary data.

## Your Task

Explain the issue and provide a secure solution.

## Solution

A read-only root filesystem prevents writes to the container's writable root filesystem.

If the process needs temporary writable storage, provide an explicit `emptyDir`:

```yaml
volumes:
- name: tmp
  emptyDir: {}
```

Mount it:

```yaml
volumeMounts:
- name: tmp
  mountPath: /tmp
```

Security principle:

```text
Root filesystem
      |
   READ ONLY
      |
      v
Explicit writable volume
      |
   only where needed
```

This is safer than making the entire root filesystem writable.

---

# Question 12 — Verify Container Security

## Scenario

You created a hardened logging sidecar.

## Your Task

Find commands to verify:

- User
- Filesystem behavior
- Container logs
- Pod configuration

## Useful Commands

Check Pod YAML:

```bash
kubectl get pod secure-logger -n security-lab -o yaml
```

Check container logs:

```bash
kubectl logs secure-logger -n security-lab -c log-agent
```

Check the running user:

```bash
kubectl exec secure-logger -n security-lab -c log-agent -- id
```

Check mounted directories:

```bash
kubectl exec secure-logger -n security-lab -c log-agent -- mount
```

Check the application log:

```bash
kubectl exec secure-logger -n security-lab -c log-agent -- ls -l /var/log/app
```

Check Pod events:

```bash
kubectl describe pod secure-logger -n security-lab
```

---

# Final Exam Challenge

## Scenario

Create a Pod:

```text
secure-logger
```

Namespace:

```text
cks-lab
```

It must contain:

```text
app
log-agent
```

Requirements:

### Application

- Image: `busybox:1.36`
- Write a timestamp to `/var/log/app/app.log`
- Write every 3 seconds.

### Logging Sidecar

- Image: `busybox:1.36`
- Use `tail -F /var/log/app/app.log`
- Mount the shared volume read-only.
- Run as non-root.
- Use a non-root UID.
- Disable privilege escalation.
- Drop all capabilities.
- Use a read-only root filesystem.

### Storage

Both containers share:

```text
emptyDir
```

Volume name:

```text
logs
```

Mount path:

```text
/var/log/app
```

## Target Architecture

```text
                  secure-logger
                       Pod
                        |
          +-------------+-------------+
          |                           |
          v                           v
        app                       log-agent
          |                           |
    read/write                    read-only
          |                           |
          +-------------+-------------+
                        |
                     emptyDir
                        |
                      app.log
```

## Expected Security Controls

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL
```

Sidecar volume:

```yaml
volumeMounts:
- name: logs
  mountPath: /var/log/app
  readOnly: true
```

---

# Quick Revision Cheat Sheet

## Multi-Container Pod

```yaml
spec:
  containers:
  - name: app
    image: busybox

  - name: sidecar
    image: busybox
```

## Shared emptyDir

```yaml
volumes:
- name: logs
  emptyDir: {}
```

## Volume Mount

```yaml
volumeMounts:
- name: logs
  mountPath: /var/log/app
```

## Read-Only Mount

```yaml
volumeMounts:
- name: logs
  mountPath: /var/log/app
  readOnly: true
```

## Sidecar Log Command

```bash
tail -F /var/log/app/app.log
```

## Container Logs

```bash
kubectl logs <pod> -c <container>
```

## Execute in Specific Container

```bash
kubectl exec -it <pod> -c <container> -- sh
```

## Non-Root

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

## Disable Privilege Escalation

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

## Read-Only Root Filesystem

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

## Drop Capabilities

```yaml
securityContext:
  capabilities:
    drop:
    - ALL
```

## Native Sidecar

```yaml
initContainers:
- name: log-agent
  image: busybox:1.36
  restartPolicy: Always
```

## Same-Pod Networking

```text
Same Pod
   |
   +-- Container A
   |
   +-- Container B
          |
       localhost
```

Containers in the same Pod share the network namespace.

---

# CKA / CKS Exam Checklist

Before moving to the next topic, make sure you can do these without notes:

- [ ] Create a multi-container Pod
- [ ] Create a shared `emptyDir`
- [ ] Mount one volume into multiple containers
- [ ] Build a logging sidecar
- [ ] Use `tail -F`
- [ ] Read logs with `kubectl logs -c`
- [ ] Use `kubectl exec -c`
- [ ] Explain `localhost` inside a Pod
- [ ] Troubleshoot mismatched volume names
- [ ] Configure a native sidecar
- [ ] Use container-level `securityContext`
- [ ] Configure `runAsNonRoot`
- [ ] Configure `runAsUser`
- [ ] Disable privilege escalation
- [ ] Drop all capabilities
- [ ] Configure a read-only root filesystem
- [ ] Configure a read-only volume mount
- [ ] Add an explicit writable `emptyDir` when required
- [ ] Troubleshoot a hardened sidecar

---

# Recommended Practice Method

For every question:

1. Read only the requirements.
2. Do not look at the solution.
3. Create the YAML yourself.
4. Apply it.
5. Verify with `kubectl`.
6. Intentionally introduce one mistake.
7. Troubleshoot it.
8. Compare your solution with the answer.
9. Repeat until you can complete it from memory.

## Most Important CKA/CKS Habit

Do not memorize only YAML.

Understand the relationship:

```text
Requirement
    ↓
Kubernetes object
    ↓
Field in YAML
    ↓
kubectl command
    ↓
Verification
    ↓
Troubleshooting
```

That workflow is more valuable than memorizing individual manifests.

---

# End of Workbook

**Topic:** Multi-Container Pods, Sidecars, Shared Volumes & Container Security

**Focus:** CKA + CKS hands-on practice

**Practice style:** Scenario-based, command-driven, troubleshooting-oriented
