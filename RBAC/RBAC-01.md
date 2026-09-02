🔐 CKA RBAC — Theory You Must Know
1. What is RBAC?

RBAC = Role-Based Access Control

It controls:

WHO can do WHAT on WHICH Kubernetes resources

Think:

Subject → Role → Permissions → Resource

Example:

dev-sa
   ↓
RoleBinding
   ↓
pod-manager
   ↓
get/list/create/delete
   ↓
pods
2. Four Important RBAC Objects

For CKA, remember these:

Object	Scope	Purpose
Role	Namespace	Permissions inside one namespace
RoleBinding	Namespace	Connects subject to Role/ClusterRole
ClusterRole	Cluster	Permissions for cluster-scoped resources or reusable permissions
ClusterRoleBinding	Cluster	Grants ClusterRole across the cluster
Easy memory trick
Role              → Namespace
RoleBinding       → Namespace
ClusterRole       → Cluster
ClusterRoleBinding → Cluster
3. Role

A Role defines permissions inside a particular namespace.

Example concept:

Role: pod-manager
Namespace: exercise-04

Allowed:
pods → get, list, watch, create, delete
services → get, list

Important:

A Role doesn't give permissions by itself.

You need a RoleBinding.

Role
 +
RoleBinding
 =
Access
4. RoleBinding ⭐⭐⭐

RoleBinding connects a user/service account/group to a Role.

Example:

ServiceAccount: dev-sa
        ↓
RoleBinding: dev-pod-access
        ↓
Role: pod-manager

This gives dev-sa the permissions defined in pod-manager.

Important CKA point

A RoleBinding can reference:

Role
ClusterRole

For example:

RoleBinding → ClusterRole

is valid.

But the resulting permissions are generally limited to the namespace of the RoleBinding.

5. ServiceAccount

A ServiceAccount represents an identity used by workloads or Kubernetes API clients.

Example:

dev-sa

Inside namespace:

exercise-04

The full identity is:

system:serviceaccount:exercise-04:dev-sa
⭐ Memorize this

For kubectl auth can-i:

--as=system:serviceaccount:<namespace>:<serviceaccount>

Example:

--as=system:serviceaccount:exercise-04:dev-sa
6. ClusterRole ⭐⭐⭐

ClusterRole defines permissions at cluster level.

Common cluster-scoped resources:

nodes
persistentvolumes
namespaces
clusterroles
clusterrolebindings

Example:

ClusterRole: node-viewer

nodes:
  get
  list

Because nodes are cluster-scoped, you need a ClusterRole/ClusterRoleBinding for cluster-wide node access.

7. ClusterRoleBinding ⭐⭐⭐

ClusterRoleBinding connects a subject to a ClusterRole across the cluster.

Example:

dev-sa
   ↓
ClusterRoleBinding
   ↓
node-viewer
   ↓
nodes → get/list

Therefore:

kubectl auth can-i list nodes \
  --as=system:serviceaccount:exercise-04:dev-sa

returns:

yes
8. API Groups ⭐⭐⭐

This is a very important CKA point.

Core API resources

Pods, Services, ConfigMaps, Secrets:

apiGroups:
- ""

NOT:

apiGroups:
- v1

Remember:

Core API = empty string

Apps API

Deployments, StatefulSets, DaemonSets:

apiGroups:
- apps
Batch API

Jobs and CronJobs:

apiGroups:
- batch
9. Resources + Verbs

RBAC permission has two major pieces:

Resource + Verb

Example:

pods + get
pods + list
pods + create
pods + delete

Common verbs:

get
list
watch
create
update
patch
delete
deletecollection
Easy understanding
Verb	Meaning
get	Read one object
list	Read multiple objects
watch	Watch changes
create	Create
update	Replace/update
patch	Partial update
delete	Delete
10. get vs list ⭐

This is commonly tested.

get pods/pod1

means:

Read a specific Pod.

list pods

means:

List Pods.

Having get doesn't automatically mean you have list.

11. Namespace Scope ⭐⭐⭐

Suppose:

Role → exercise-04

and:

RoleBinding → exercise-04

The ServiceAccount can access resources according to that Role in exercise-04.

It doesn't automatically get the same permission in:

default
production
qa

That's why:

kubectl auth can-i list pods \
  -n exercise-04 \
  --as=system:serviceaccount:exercise-04:dev-sa

can return:

yes

while:

kubectl auth can-i list pods \
  -n default \
  --as=system:serviceaccount:exercise-04:dev-sa

returns:

no
12. kubectl auth can-i ⭐⭐⭐⭐⭐

For CKA, memorize this command.

It answers:

Can this user/service account perform this action?

Syntax:

kubectl auth can-i <verb> <resource>

Example:

kubectl auth can-i list pods -n exercise-04

For ServiceAccount:

kubectl auth can-i list pods \
  -n exercise-04 \
  --as=system:serviceaccount:exercise-04:dev-sa

Output:

yes

or:

no
13. Debugging RBAC ⭐⭐⭐⭐⭐

If someone says:

"I cannot delete the Deployment."

Don't immediately modify RBAC.

First:

kubectl auth can-i delete deployments \
  -n exercise-04 \
  --as=system:serviceaccount:exercise-04:dev-sa

If:

no

check the Role:

kubectl get role pod-manager -n exercise-04 -o yaml

Then check RoleBinding:

kubectl get rolebinding dev-pod-access \
  -n exercise-04 -o yaml
14. --list ⭐⭐⭐⭐

To see permissions:

kubectl auth can-i --list \
  -n exercise-04 \
  --as=system:serviceaccount:exercise-04:dev-sa

This is extremely useful during the CKA.

15. RBAC Subresources ⭐⭐⭐

This is a slightly advanced but important concept.

Examples:

pods
pods/log
pods/exec
deployments/scale

They can require explicit permissions.

For example:

pods/log

is different from simply thinking:

pods

And scaling a Deployment involves:

deployments/scale

with appropriate verbs such as:

patch
CKA memory
deployment
deployment/scale

pod
pod/log
pod/exec

Think:

Resource/Subresource can have separate authorization requirements.

🧠 CKA RBAC Cheat Sheet

Memorize this:

RBAC
│
├── Role
│    └── Namespace-scoped permissions
│
├── RoleBinding
│    └── Connect Subject → Role
│
├── ClusterRole
│    └── Cluster-level/reusable permissions
│
└── ClusterRoleBinding
     └── Connect Subject → ClusterRole
Most important commands
kubectl create sa dev-sa -n exercise-04
kubectl create role pod-manager \
  --verb=get,list,watch,create,delete \
  --resource=pods \
  -n exercise-04
kubectl create rolebinding dev-pod-access \
  --role=pod-manager \
  --serviceaccount=exercise-04:dev-sa \
  -n exercise-04
kubectl create clusterrole node-viewer \
  --verb=get,list \
  --resource=nodes
kubectl create clusterrolebinding dev-node-access \
  --clusterrole=node-viewer \
  --serviceaccount=exercise-04:dev-sa
⭐⭐⭐⭐⭐ Most important exam command
kubectl auth can-i

Remember:

RBAC = Who + Verb + Resource + Scope

That's the core theory you need for the CKA RBAC questions.
---
Yes 👍 Let's continue with RBAC — next CKA topic: RoleBinding vs ClusterRoleBinding.

1. RoleBinding vs ClusterRoleBinding ⭐⭐⭐⭐⭐

This is very important for CKA.

RoleBinding

A RoleBinding grants permissions within a namespace.

ServiceAccount
      ↓
 RoleBinding
      ↓
    Role
      ↓
 namespace

Example:

kind: RoleBinding
metadata:
  name: dev-access
  namespace: qa
subjects:
- kind: ServiceAccount
  name: dev-sa
  namespace: qa
roleRef:
  kind: Role
  name: pod-manager
  apiGroup: rbac.authorization.k8s.io

This means:

dev-sa gets the permissions of pod-manager in qa.

It does not automatically give access to default.

2. ClusterRoleBinding

A ClusterRoleBinding grants a ClusterRole's permissions cluster-wide.

ServiceAccount
      ↓
ClusterRoleBinding
      ↓
 ClusterRole
      ↓
   Cluster

Example:

kind: ClusterRoleBinding
metadata:
  name: dev-node-access
subjects:
- kind: ServiceAccount
  name: dev-sa
  namespace: qa
roleRef:
  kind: ClusterRole
  name: node-viewer
  apiGroup: rbac.authorization.k8s.io

Now:

kubectl auth can-i list nodes \
  --as=system:serviceaccount:qa:dev-sa

➡️ yes

because nodes are cluster-scoped.

⭐ Important CKA Scenario

Suppose you have:

Role: pod-reader
Namespace: qa

RoleBinding: dev-access
Namespace: qa

ServiceAccount: dev-sa
Namespace: qa

Then:

kubectl auth can-i list pods -n qa \
  --as=system:serviceaccount:qa:dev-sa

✅ yes

But:

kubectl auth can-i list pods -n default \
  --as=system:serviceaccount:qa:dev-sa

❌ no

3. Very Important Trick

A ClusterRole can be used with a RoleBinding.

For example:

ClusterRole
   ↓
RoleBinding
   ↓
ServiceAccount

Even though the ClusterRole exists cluster-wide, the RoleBinding limits the granted permissions to the RoleBinding's namespace.

This is a common CKA concept.

4. ClusterRole vs Role — Easy Memory
Role
Role
 ↓
Namespace resources
 ↓
pods, services, deployments...
ClusterRole
ClusterRole
 ↓
Cluster-scoped resources
 ↓
nodes, PVs, namespaces...

But remember:

ClusterRole can also define permissions for namespaced resources and be reused through RoleBindings.

5. CKA Must-Know Commands

Check permission:

kubectl auth can-i list pods -n qa \
  --as=system:serviceaccount:qa:dev-sa

Check cluster permission:

kubectl auth can-i list nodes \
  --as=system:serviceaccount:qa:dev-sa

See all permissions:

kubectl auth can-i --list -n qa \
  --as=system:serviceaccount:qa:dev-sa

Inspect Role:

kubectl get role pod-manager -n qa -o yaml

Inspect RoleBinding:

kubectl get rolebinding dev-access -n qa -o yaml

Inspect ClusterRole:

kubectl get clusterrole node-viewer -o yaml

Inspect ClusterRoleBinding:

kubectl get clusterrolebinding dev-node-access -o yaml
🧠 Remember this for CKA

Role = permissions
RoleBinding = connects permissions to a subject
ClusterRole = cluster-level/reusable permissions
ClusterRoleBinding = cluster-wide connection
auth can-i = verify/debug permissions

Next RBAC topic

After this, we should cover RBAC API Groups + Resources + Verbs, followed by RBAC troubleshooting with kubectl auth can-i. These are highly useful for CKA.

---



