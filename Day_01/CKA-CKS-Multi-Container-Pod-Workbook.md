# CKA/CKS Multi-Container Pod & Sidecar Workbook

A hands-on practice workbook combining **CKA workload/container skills** with
**CKS container-hardening (security) skills**, using the multi-container
sidecar logging pattern as the running example.

Each scenario follows the exam-style workflow:

> **Question → Your Task → Hints → Solution → Explanation → Troubleshooting**

---

## How to Use This Workbook

1. Read the **Question** and attempt the **Your Task** section yourself first,
   in a real or practice cluster (`kind`, `minikube`, or killer.sh style env).
2. Only look at **Hints** if you get stuck.
3. Compare your YAML against **Solution**.
4. Read **Explanation** to understand *why*, not just *what*.
5. Use **Troubleshooting** as a checklist if your Pod doesn't behave as expected.

Recommended flow: Level 1 → Level 2 → Level 3 (CKA fundamentals before CKS hardening).

---

## Part A — CKA Focus: Multi-Container Pods & Sidecars

### Question 1 — Sidecar Logging (Basic)

**Scenario:** You need a Pod where an app container writes logs to a file,
and a sidecar container tails that file continuously.

**Your Task**

Create a Pod named `secure-logger` in namespace `security-lab`.

- Main container: `app`, image `busybox:1.36`
  - Writes `application started` to `/var/log/app/app.log` every 5 seconds.
- Sidecar container: `log-agent`, image `busybox:1.36`
  - Continuously reads `/var/log/app/app.log` using `tail -F`.
- Both containers share an `emptyDir` volume called `log-volume`.

**Hints**

- You'll need `containers:` (not `initContainers:`) for both.
- Use a shell loop (`while true; do ... ; sleep 5; done`) for the writer.
- Both containers need identical `mountPath` values so they see the same file.
- Namespace must exist first: `kubectl create ns security-lab`.

**Solution**

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
    command: ["sh", "-c"]
    args:
      - >
        while true; do
          echo "application started" >> /var/log/app/app.log;
          sleep 5;
        done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  - name: log-agent
    image: busybox:1.36
    command: ["sh", "-c", "tail -F /var/log/app/app.log"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

**Explanation**

- `emptyDir` creates a temporary directory on the node, shared by every
  container in the Pod that mounts it — this is the simplest way for
  sidecars to exchange files.
- Both containers must mount the **same volume name** at a path where the
  file actually lands (`/var/log/app/app.log`), otherwise they're looking
  at two different, unconnected directories.
- `tail -F` (capital F) follows the file even if it's rotated/recreated —
  more robust than lowercase `-f` for production sidecars.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| `log-agent` container crashes immediately | File doesn't exist yet when `tail -F` starts — `-F` handles this, `-f` would not |
| Sidecar shows no output | `mountPath` differs between containers, or `volumes.name` doesn't match `volumeMounts.name` |
| `app` container in `CrashLoopBackOff` | Missing `while true` loop — command exits after first run |

---

### Question 2 — Native Sidecar (initContainer with restartPolicy: Always)

**Scenario:** Kubernetes 1.28+ supports "native sidecars" — containers
defined under `initContainers` with `restartPolicy: Always`, which start
before the main container and keep running alongside it.

**Your Task**

Create a Pod called `logger` in namespace `cka-lab`.

- Native sidecar: `log-agent` (defined in `initContainers`, image
  `busybox:1.36`, `restartPolicy: Always`), running `tail -F /var/log/app/app.log`.
- Main container: `app`, image `busybox:1.36`, writing a timestamp to
  `/var/log/app/app.log` every 3 seconds.
- Both share an `emptyDir`.

Then answer: **what command verifies the sidecar's logs?**

**Hints**

- Native sidecars live under `initContainers:`, not `containers:`.
- Without `restartPolicy: Always` on that entry, it's a normal init
  container — it would run once and exit, blocking the main container forever.
- `kubectl logs <pod> -c <container>` works the same for init and regular containers.

**Solution**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
  namespace: cka-lab
spec:
  initContainers:
  - name: log-agent
    image: busybox:1.36
    restartPolicy: Always
    command: ["sh", "-c", "tail -F /var/log/app/app.log"]
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  containers:
  - name: app
    image: busybox:1.36
    command: ["sh", "-c"]
    args:
      - >
        while true; do
          date >> /var/log/app/app.log;
          sleep 3;
        done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

**Verification command**

```bash
kubectl logs logger -n cka-lab -c log-agent
```

**Explanation**

- `restartPolicy: Always` on an init container tells the kubelet: "start
  this before the main containers, then keep it running (and restart it if
  it dies) for the life of the Pod" — that's what makes it a *native
  sidecar* instead of a one-shot init step.
- Native sidecars are guaranteed to start before the main container and are
  terminated *after* main containers on Pod shutdown — useful for logging,
  proxies, and service mesh sidecars.
- `kubectl logs` treats sidecars exactly like regular containers once
  they're running — always pass `-c <name>` for multi-container Pods.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| Main container never starts | Forgot `restartPolicy: Always` — sidecar ran once, exited, and (as a normal init container) blocked the rest of the Pod |
| `kubectl logs` says "a container name must be specified" | Pod has multiple containers — you must pass `-c` |
| Sidecar restarts repeatedly | Command exits on its own (e.g., `tail -f` without `-F` when file briefly disappears) |

---

### Question 3 — Shared Volume Troubleshooting

**Scenario:** A given Pod spec is broken — the sidecar can't see the app's log file.

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

**Your Task**

Find and fix the bug.

**Hints**

Compare all three volume name references:

```
app.volumeMounts.name       = ?
log-agent.volumeMounts.name = ?
volumes.name                 = ?
```

They must **all** match exactly.

**Solution**

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
    - name: log-volume   # fixed: was "logs"
      mountPath: /var/log/app

  volumes:
  - name: log-volume
    emptyDir: {}
```

**Explanation**

`volumeMounts.name` is just a *reference* to an entry under `volumes:` — it
is matched by string, not by mountPath. `log-agent` referenced a volume
called `logs`, which doesn't exist in `volumes:`, so the Pod would actually
fail to schedule (`Error: configmap "logs" not found`-style error, or more
precisely a volume-not-found validation error) — not silently mount an
empty directory.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| Pod fails validation / won't create | `volumeMounts.name` references a `volumes` entry that doesn't exist |
| Sidecar directory is empty | Mount paths match but volume names differ — each container ends up with its own private `emptyDir` (only possible if both names *do* exist under `volumes:`, just as separate volumes) |
| `kubectl describe pod` shows no clear error | Check `kubectl get pod -o yaml` and diff volume names manually — this is the single most common multi-container bug |

---

### Question 4 — Same-Pod Networking (localhost)

**Scenario:** A Pod `web-test` has two containers: `web` (nginx) and
`tester`, which must reach nginx via `localhost:80`.

**Your Task**

Explain **why** `tester` can reach nginx via `localhost:80` instead of a Pod IP or DNS name.

**Hints**

Think about what a Pod actually *is* at the OS/kernel level — it's not "two VMs," it's one network namespace shared by multiple containers.

**Solution (YAML)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-test
spec:
  containers:
  - name: web
    image: nginx

  - name: tester
    image: busybox
    command: ["sh", "-c", "while true; do wget -qO- localhost:80; sleep 5; done"]
```

**Expected concept answer**

> Because containers in the same Pod share the network namespace.

```
Pod
├── web     (listens on port 80)
└── tester  (connects to localhost:80)
      ↓
shared network namespace
      ↓
same loopback interface, same IP, same port space
```

**Explanation**

- All containers in a Pod share one network namespace, one IP address, and
  one loopback interface (`localhost`/`127.0.0.1`).
- This means containers **cannot** both bind the same port (e.g., two
  containers listening on `:80` will conflict) — port space is shared, not per-container.
- This is exactly why sidecar proxies (Envoy, Istio) can intercept traffic
  to `localhost` — they live in the same network namespace as your app.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| `wget: bad address` | Typo in port/host, or nginx container not yet ready |
| Connection refused | nginx container still starting — add a retry loop or `sleep` before first request |
| Port already in use | Two containers in the Pod tried to bind the same port — not allowed since they share the network namespace |

---

## Part B — CKS Focus: Container Hardening

### Question 5 — Read-Only Volume Mount

**Scenario (CKS security requirement):** *"The logging sidecar must not be
able to modify application files."* The app writes `/var/log/app/app.log`;
the sidecar only reads it.

**Your Task**

Make the sidecar's mount of the shared volume read-only.

**Hints**

- `readOnly` is a field on `volumeMounts`, not on `volumes`.
- Only set it on the **sidecar's** mount — the app still needs to write.

**Solution**

```yaml
containers:
- name: log-agent
  volumeMounts:
  - name: log-volume
    mountPath: /var/log/app
    readOnly: true
```

**Explanation**

```
Application
    │ read/write
    ▼
 emptyDir  ← shared volume
    ▲
    │ read-only
Sidecar
```

`readOnly: true` mounts the volume read-only *inside that specific
container's filesystem view* — even though the underlying `emptyDir` is
writable, the sidecar's kernel-level mount is enforced as read-only, so
any `write()` syscall from that container fails with `EROFS`
(read-only file system), regardless of what the process tries to do.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| App container also can't write | `readOnly: true` mistakenly applied to the app's `volumeMounts` entry instead of the sidecar's |
| Sidecar still writes files fine | `readOnly` set at wrong nesting level, or applied to a different volume than the one actually mounted at that path |

---

### Question 6 — Least Privilege (Non-Root + No Privilege Escalation)

**Scenario:** A security scan finds `log-agent` running as root.

**Your Task**

Modify `log-agent` so it:

- does not run as root
- uses a non-root user
- has privilege escalation disabled
- uses a read-only root filesystem

**Hints**

- `runAsNonRoot: true` alone just *validates* the container doesn't run as
  root — you usually pair it with an explicit `runAsUser`.
- `allowPrivilegeEscalation: false` blocks `setuid`-style privilege gains
  even if the process tries to escalate.
- `readOnlyRootFilesystem` locks the container's `/` — remember any
  directories the process needs to write to (e.g. `/tmp`) may need an
  explicit `emptyDir` mount.

**Solution**

```yaml
containers:
- name: log-agent
  image: busybox:1.36
  command: ["sh", "-c", "tail -F /var/log/app/app.log"]
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
  volumeMounts:
  - name: log-volume
    mountPath: /var/log/app
    readOnly: true
```

**Explanation**

| Field | Effect |
|---|---|
| `runAsNonRoot: true` | kubelet refuses to start the container if the resolved user is UID 0 |
| `runAsUser: 1000` | explicitly picks a non-root UID so scheduling doesn't fail on an image whose default user *is* root |
| `allowPrivilegeEscalation: false` | sets `no_new_privs`, blocking `setuid`/`setgid` binaries from gaining more privileges than the parent process |
| `readOnlyRootFilesystem: true` | mounts the container's root filesystem read-only, limiting the blast radius of a compromised process |

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| `CreateContainerConfigError` / `container has runAsNonRoot and image will run as root` | Image's default user is root and no `runAsUser` was set |
| Container crashes on startup writing to `/tmp` or cache dirs | `readOnlyRootFilesystem: true` blocks writes — mount an `emptyDir` at that path if the process needs scratch space |
| Sidecar can't read the log at all | `runAsUser` UID doesn't have read permission on the file — check file ownership vs. the UID chosen |

---

### Question 7 — Per-Container securityContext Placement

**Scenario:**

```yaml
containers:
- name: app
  image: nginx
- name: log-agent
  image: busybox
```

Requirement: *"Only `log-agent` must run as non-root. Do not change the
security settings of the application container."*

**Your Task**

Decide: Pod-level `securityContext`, or container-level?

**Hints**

- Pod-level `securityContext` applies to **all** containers unless
  overridden — that would violate "do not change the app container."
- Container-level settings override Pod-level ones for that container only.

**Solution**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mixed-security-pod
spec:
  containers:
  - name: app
    image: nginx
    # unchanged — no securityContext added here

  - name: log-agent
    image: busybox
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
```

**Explanation**

```
Pod-level securityContext   → applies to every container (default/floor)
Container-level securityContext → overrides Pod-level, scoped to one container
```

Because the requirement is scoped to a *single* container, the setting
belongs at the **container level**. Setting it at the Pod level would
force nginx to also run as non-root, which the requirement explicitly forbids.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| nginx container fails to start after your change | Accidentally applied `runAsNonRoot` at Pod level, affecting nginx too |
| Exam grader marks it wrong even though the Pod runs | Correct behavior isn't just "Pod works" — the *scope* of the setting matters; double check YAML structure, not just runtime result |

---

### Question 8 — Dropping Linux Capabilities

**Scenario:** A security scan reports `log-agent` has unnecessary Linux capabilities.

**Your Task**

Drop all Linux capabilities from the sidecar.

**Hints**

- `capabilities` lives under `securityContext`, as a nested `drop`/`add` list.
- `drop: ["ALL"]` removes every default capability; only add back specific
  ones (e.g., `NET_BIND_SERVICE`) if the container truly needs them.

**Solution**

```yaml
containers:
- name: log-agent
  securityContext:
    capabilities:
      drop:
      - ALL
```

**CKS memory aid**

```
Unnecessary privileges
        ↓
   Drop capabilities
        ↓
securityContext:
  capabilities:
    drop:
    - ALL
```

**Explanation**

Container runtimes grant a default set of Linux capabilities (e.g.,
`CHOWN`, `NET_RAW`, `SETUID`) even to "normal" containers. For a sidecar
that only reads a file, none of these are needed. Dropping `ALL` strips
every default capability, following the principle of least privilege — if
the sidecar is ever compromised, it has far fewer kernel-level abilities
to abuse.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| Sidecar can't bind to a low port (<1024) | Needed `NET_BIND_SERVICE` was dropped along with everything else — add it back explicitly if required |
| YAML rejected / capability name error | Capability names must be UPPERCASE strings (e.g. `ALL`, `NET_ADMIN`), not lowercase |

---

### Question 9 — Combined CKS Hardening Scenario

**Scenario:** Full hardening exercise combining everything above.

**Your Task**

Create Pod `secure-logger` in namespace `security-lab`:

- `app` (image `nginx`) and `log-agent` (image `busybox`)
- Share an `emptyDir`
- `log-agent` reads the app's logs
- `log-agent`'s volume mount is read-only
- `log-agent` runs as non-root
- privilege escalation disabled
- all Linux capabilities dropped
- read-only root filesystem

**Security checklist**

- [x] Multi-container Pod
- [x] Shared `emptyDir`
- [x] `readOnly` volume mount
- [x] `runAsNonRoot`
- [x] `allowPrivilegeEscalation: false`
- [x] `capabilities.drop: [ALL]`
- [x] `readOnlyRootFilesystem`

**Solution**

```yaml
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
    - name: log-volume
      mountPath: /var/log/app

  - name: log-agent
    image: busybox
    command: ["sh", "-c", "tail -F /var/log/app/app.log"]
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/app
      readOnly: true

  volumes:
  - name: log-volume
    emptyDir: {}
```

**Explanation**

This ties together CKA workload knowledge (multi-container Pod, shared
`emptyDir`, correct volume references) with CKS hardening (least-privilege
`securityContext` scoped only to the sidecar that needs it). Notice `app`
(nginx) is left with default security settings — nginx typically needs to
bind to port 80 and may need root or specific capabilities depending on
the image, so hardening is applied surgically to the container that can
tolerate it.

**Troubleshooting**

| Symptom | Likely Cause |
|---|---|
| `log-agent` never becomes Ready | `readOnlyRootFilesystem` blocks something it needs to write (e.g., shell temp files) — verify busybox's `tail -F` doesn't need scratch space, or mount a small writable `emptyDir` at the specific path required |
| App container also hardened unexpectedly | securityContext accidentally placed at Pod level instead of only under `log-agent` |
| `CreateContainerConfigError` | `runAsNonRoot: true` set without a valid `runAsUser`, and busybox's default user is root |

---

## Part C — Final Practice Challenge (No Solution Shown)

> Create a Pod named `secure-logger` in namespace `cks-lab`. It must contain
> an application container and a logging sidecar. Both containers share an
> `emptyDir` volume named `logs`. The application writes logs to
> `/var/log/app/app.log`. The sidecar uses `tail -F` to read the file. The
> sidecar's volume mount must be read-only. The sidecar must run as
> non-root, disable privilege escalation, drop all capabilities, and use a
> read-only root filesystem.

```
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
               emptyDir (logs)
                  │
              app.log
```

Try this fully from memory before checking back against Question 9's pattern.

---

## Cheat Sheet — CKA vs CKS Mental Model

**CKA asks:** *Can you create and troubleshoot the Pod?*

- Pod / multi-container Pod structure
- `initContainers` (including native sidecars with `restartPolicy: Always`)
- `emptyDir` and `volumeMounts`
- Same-Pod networking via `localhost`
- `kubectl logs -c <container>`

**CKS asks:** *Can you secure the Pod/container?*

- `securityContext` — Pod-level vs container-level scoping
- `runAsNonRoot` / `runAsUser`
- `allowPrivilegeEscalation: false`
- `capabilities.drop: [ALL]`
- `readOnlyRootFilesystem: true`
- `readOnly: true` on `volumeMounts`
- Broader Pod Security (Admission) standards

### Quick Revision Commands

```bash
# Create namespace
kubectl create ns security-lab

# Apply a Pod spec
kubectl apply -f secure-logger.yaml

# Check status of all containers in a Pod
kubectl get pod secure-logger -n security-lab -o wide

# View logs from a specific container
kubectl logs secure-logger -n security-lab -c log-agent

# Describe for volume/mount/securityContext errors
kubectl describe pod secure-logger -n security-lab

# Exec into a specific container to verify UID / filesystem
kubectl exec -it secure-logger -n security-lab -c log-agent -- id
kubectl exec -it secure-logger -n security-lab -c log-agent -- sh -c "touch /test || echo READ-ONLY CONFIRMED"

# Dry-run validate YAML without creating
kubectl apply -f secure-logger.yaml --dry-run=client -o yaml
```
