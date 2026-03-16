Kubernetes Quick Revision Sheet (DevOps Interview – Uday Patil)
1. CoreDNS
What is CoreDNS

CoreDNS is the DNS server inside Kubernetes responsible for resolving service names to cluster IP addresses.

Pods use DNS instead of IPs to communicate.

Example:

frontend → backend.default.svc.cluster.local
Where CoreDNS Runs

Namespace: kube-system

Resource Type: Deployment

Check:

kubectl get pods -n kube-system | grep coredns
DNS Resolution Flow
Pod
 │
 │ DNS Query
 ▼
CoreDNS
 │
 ▼
Service ClusterIP
 │
 ▼
kube-proxy routes to Pod
Troubleshooting DNS Issues

Check CoreDNS pods

kubectl get pods -n kube-system

Check CoreDNS service

kubectl get svc -n kube-system

Check DNS from pod

kubectl exec -it pod -- nslookup kubernetes.default
Ready-to-Speak Answer

CoreDNS is the DNS server used inside Kubernetes clusters to resolve service names to their cluster IP addresses. It runs as a deployment in the kube-system namespace and allows pods to communicate using DNS instead of dynamic IP addresses.

2. kube-proxy
What is kube-proxy

kube-proxy is a network component that runs on every node and manages service networking.

It forwards traffic from:

Service → Pod
Where kube-proxy Runs

Resource Type:

DaemonSet

Meaning:

1 kube-proxy pod per node
kube-proxy Modes
Mode	Description
iptables	Default routing rules
IPVS	High-performance load balancing
Userspace	Deprecated
Traffic Flow
Client Pod
     │
     ▼
Service ClusterIP
     │
     ▼
kube-proxy
     │
     ▼
Backend Pods
Ready-to-Speak Answer

kube-proxy is a Kubernetes networking component that runs on each node and maintains network rules to ensure traffic sent to a service IP is correctly forwarded to backend pods.

3. CNI (Container Network Interface)
What is CNI

CNI is responsible for:

Assigning IP addresses to pods

Enabling pod-to-pod networking

Common CNI Plugins
CNI	Environment
AWS VPC CNI	EKS
Calico	Networking + NetworkPolicy
Flannel	Simple clusters
Cilium	Advanced networking
AWS VPC CNI Behavior

Pods receive VPC IP addresses.

Example:

Node IP: 10.0.1.10
Pods:
10.0.1.21
10.0.1.22
Real Production Issue

Pods stuck in Pending state.

Root Cause:

Subnet ran out of IP addresses

Check logs:

kubectl logs aws-node -n kube-system
Ready-to-Speak Answer

CNI is responsible for pod networking in Kubernetes. It assigns IP addresses to pods and enables communication between pods across nodes.

4. Metrics Server
What is Metrics Server

Metrics Server collects CPU and memory metrics from kubelet.

Used by:

kubectl top
HPA (Horizontal Pod Autoscaler)
Architecture
Pod Metrics
   │
   ▼
kubelet
   │
   ▼
Metrics Server
   │
   ▼
API Server
   │
   ▼
kubectl top / HPA
Commands

Check metrics:

kubectl top pods
kubectl top nodes

Check metrics server:

kubectl get pods -n kube-system | grep metrics-server
Ready-to-Speak Answer

Metrics Server collects CPU and memory usage from kubelets on each node and exposes them through the Kubernetes Metrics API, which is used by commands like kubectl top and by the Horizontal Pod Autoscaler.

5. Authentication vs Authorization
Authentication

Verifies who the user is.

Examples:

certificates

tokens

IAM (EKS)

Authorization

Verifies what actions the user can perform.

Implemented using RBAC.

Flow
Request
 │
 ▼
Authentication
 │
 ▼
Authorization (RBAC)
 │
 ▼
Admission Controllers
 │
 ▼
etcd
Ready-to-Speak Answer

Authentication verifies the identity of the user, while authorization determines whether that user has permission to perform the requested action using RBAC policies.

6. RBAC

RBAC controls access to Kubernetes resources.

RBAC Components
Resource	Purpose
Role	namespace permissions
ClusterRole	cluster-wide permissions
RoleBinding	attach role to user
ClusterRoleBinding	attach cluster role
Example

Allow pod read access:

kind: Role
rules:
- resources: ["pods"]
  verbs: ["get","list","watch"]
Debug RBAC
kubectl auth can-i get pods
Ready-to-Speak Answer

RBAC is Kubernetes' authorization mechanism used to control which users or service accounts can perform actions on cluster resources.

7. Service Accounts

Service Accounts provide identity for pods.

Pods authenticate to API server using service account tokens.

Token path inside pod:

/var/run/secrets/kubernetes.io/serviceaccount
Example
serviceAccountName: app-sa
Ready-to-Speak Answer

Service accounts provide identities for applications running inside pods and allow them to authenticate with the Kubernetes API.

8. Admission Controllers

Admission controllers validate or modify requests before storing them in etcd.

Types
Type	Purpose
Mutating	modifies requests
Validating	validates requests
Examples

PodSecurity

LimitRange

ResourceQuota

Ready-to-Speak Answer

Admission controllers intercept API requests after authentication and authorization to enforce policies before objects are stored in etcd.

9. Pod Security

Pod Security prevents unsafe pod configurations.

Examples blocked:

privileged containers

root containers

hostPath mounts

Pod Security Levels
Level	Security
Privileged	unrestricted
Baseline	basic security
Restricted	strict security
Ready-to-Speak Answer

Pod Security enforces security standards for pods and prevents insecure configurations such as privileged containers or host filesystem access.

10. Network Policies

NetworkPolicy controls pod-to-pod communication.

Acts like firewall rules inside Kubernetes.

Types
Policy	Description
Ingress	incoming traffic
Egress	outgoing traffic
Example

Allow frontend → backend

podSelector:
  matchLabels:
    app: backend
Ready-to-Speak Answer

NetworkPolicy is used to control pod communication by defining rules for ingress and egress traffic between pods.

11. Kubernetes Secrets

Secrets store sensitive data:

passwords

tokens

API keys

Create Secret
kubectl create secret generic db-secret \
--from-literal=password=mydbpassword
Important Note

Secrets are:

Base64 encoded
NOT encrypted

Use etcd encryption in production.

Ready-to-Speak Answer

Kubernetes Secrets store sensitive information like passwords and API keys and allow secure injection into pods via environment variables or volumes.

12. kubeconfig

kubeconfig tells kubectl how to connect to a cluster.

Location:

~/.kube/config
kubeconfig Components
Field	Purpose
cluster	API server endpoint
user	authentication credentials
context	cluster + user
Ready-to-Speak Answer

kubeconfig is a client configuration file used by kubectl to authenticate and connect to Kubernetes clusters and manage multiple cluster contexts.

13. IRSA (IAM Roles for Service Accounts)

IRSA allows pods to access AWS services securely.

Without storing AWS credentials.

IRSA Architecture
Pod
 │
 ▼
ServiceAccount
 │
 ▼
IAM Role
 │
 ▼
OIDC Provider
 │
 ▼
AWS STS
 │
 ▼
Temporary credentials
 │
 ▼
AWS Service (S3)
Service Account Annotation
eks.amazonaws.com/role-arn
Ready-to-Speak Answer

IRSA allows Kubernetes pods to securely access AWS services by associating an IAM role with a Kubernetes service account through an OIDC identity provider.

14. What Happens When You Run kubectl apply
Flow
kubectl
 │
 ▼
API Server
 │
 ├ Authentication
 ├ Authorization
 ├ Admission Controllers
 │
 ▼
etcd
 │
 ▼
Deployment Controller
 │
 ▼
ReplicaSet
 │
 ▼
Pods created
 │
 ▼
Scheduler selects node
 │
 ▼
kubelet
 │
 ▼
Container Runtime
 │
 ▼
Pod Running
Ready-to-Speak Answer

When kubectl apply is executed, the request is sent to the API server where authentication, authorization, and admission checks occur. The deployment object is stored in etcd, the controller manager creates a ReplicaSet, the scheduler assigns pods to nodes, and kubelet starts containers through the container runtime.

End of Sheet

This sheet covers all Kubernetes concepts we reviewed in this session and is suitable for:

GitHub revision notes

DevOps interviews

quick last-minute revision

If you want, I can also prepare a second sheet called

“Top 25 Kubernetes Interview Questions for 3-5 Years DevOps Engineers”

which would be perfect to keep in your GitHub repo as well.
