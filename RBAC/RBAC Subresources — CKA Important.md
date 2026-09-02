RBAC Subresources — CKA Important ⭐⭐⭐⭐

A subresource is a secondary API endpoint under a Kubernetes resource.

For example:

pods
├── pods/log
├── pods/exec
└── pods/status

and:

deployments
└── deployments/scale

The important CKA point is:

A parent resource and its subresources can require separate RBAC permissions.

1. pods/log

If you want someone to read Pod logs:

kubectl logs mypod

RBAC can use:

resources:
- pods/log

Example:

rules:
- apiGroups:
  - ""
  resources:
  - pods/log
  verbs:
  - get

Test it:

kubectl auth can-i get pods/log \
  -n qa \
  --as=system:serviceaccount:qa:dev-sa

Expected:

yes
Remember
kubectl get pods
       ↓
     pods

kubectl logs pod
       ↓
    pods/log
2. pods/exec

When you run:

kubectl exec -it mypod -- sh

you're accessing the exec subresource.

RBAC can specify:

resources:
- pods/exec

For example:

rules:
- apiGroups:
  - ""
  resources:
  - pods/exec
  verbs:
  - create

Test:

kubectl auth can-i create pods/exec \
  -n qa \
  --as=system:serviceaccount:qa:dev-sa
CKA memory
kubectl exec
     ↓
pods/exec
3. deployments/scale

Suppose you want to allow a user to scale a Deployment:

kubectl scale deployment web --replicas=3

The relevant subresource is:

deployments/scale

Example:

rules:
- apiGroups:
  - apps
  resources:
  - deployments/scale
  verbs:
  - get
  - patch

Test:

kubectl auth can-i patch deployments/scale \
  -n qa \
  --as=system:serviceaccount:qa:dev-sa
4. pods/status

Pods also have:

pods/status

Example:

rules:
- apiGroups:
  - ""
  resources:
  - pods/status
  verbs:
  - get
  - patch
  - update

You don't normally need to memorize every Pod subresource, but understand the concept.

5. Very Important: API Group Still Matters

Look carefully:

Pod subresources

Pods are Core API:

apiGroups:
- ""

So:

resources:
- pods/log

uses:

apiGroups:
- ""
Deployment subresources

Deployments belong to apps:

apiGroups:
- apps

So:

resources:
- deployments/scale

uses:

apiGroups:
- apps
6. Parent Resource ≠ Subresource

Suppose your Role contains:

rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list

This allows:

kubectl get pods

But don't assume this automatically gives permission for:

kubectl logs mypod

For exam purposes, treat:

pods
pods/log

as separate RBAC resources and explicitly grant the subresource when required.

7. Wildcard Gotcha ⭐⭐⭐

You may see:

resources:
- "*"

Don't rely on a wildcard when the exam specifically requires subresource access.

If the requirement explicitly says:

Allow access to Pod logs.

Use:

resources:
- pods/log

If it says:

Allow execution inside Pods.

Use:

resources:
- pods/exec

This makes your intended permission explicit.

8. CKA Troubleshooting Pattern ⭐⭐⭐⭐⭐

If a developer says:

"I can see Pods, but I cannot view their logs."

First check:

kubectl auth can-i get pods \
  -n qa \
  --as=system:serviceaccount:qa:dev-sa

Maybe:

yes

Then check:

kubectl auth can-i get pods/log \
  -n qa \
  --as=system:serviceaccount:qa:dev-sa

Maybe:

no

Now you know the problem:

pods      → yes
pods/log  → no

So the Role needs permission for:

resources:
- pods/log
🧠 CKA Subresource Cheat Sheet
Operation	RBAC resource	API Group
Get Pods	pods	""
Get logs	pods/log	""
Exec into Pod	pods/exec	""
Pod status	pods/status	""
Scale Deployment	deployments/scale	apps
Deployment status	deployments/status	apps
Scale StatefulSet	statefulsets/scale	apps
Memorize these 4:
pods/log
pods/exec
pods/status
deployments/scale

And remember:

Subresource = parent/subresource

For CKA, whenever you see a requirement involving logs, exec, scale, or status, stop and ask yourself: "Is this a subresource?"
