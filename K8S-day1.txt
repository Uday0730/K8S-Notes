

#### NOTES ####

1) Kubernetes architecture — core idea

Keywords: Control plane (API Server, etcd, Scheduler, Controller Manager), Worker nodes (kubelet, container runtime, kube-proxy).

Why: Control plane stores desired state in etcd; workers execute containers.

KEY: “K8s separates brain (control plane) from workers; the API server records desired state in etcd and controllers + scheduler ensure pods run on nodes.”

2) Pods / ReplicaSet / Deployment (controller + self-healing)

Keywords: Pod = smallest unit; ReplicaSet ensures N replicas; Deployment manages ReplicaSets + rollouts/rollback.

Why: Provides scaling, rolling updates, automatic recreation of failed pods.

Interview line: “Deployments provide declarative rollouts and self-healing — controllers reconcile desired vs actual state.”

Commands: kubectl apply -f deployment.yaml, kubectl scale deployment/name --replicas=N

3) Why Services? (stable endpoint & discovery)

Keywords: ClusterIP (internal), NodePort (expose node:port), LoadBalancer (cloud LB), Service = stable DNS + virtual IP.

Why: Pods are ephemeral — Service gives a stable network identity + load balancing.

Interview line: “Services decouple clients from ephemeral pod IPs by providing a stable ClusterIP/DNS and load balancing.”

Quick test: kubectl get svc; inside cluster: curl <ClusterIP>:<port>; outside: curl http://<NodeIP>:<NodePort>

4) NodePort vs LoadBalancer vs Ingress (when to use)

Keywords: NodePort = dev/demo; LoadBalancer = one LB per service (cloud); Ingress = single LB + path/host routing.

Why: Ingress reduces cost & complexity for many microservices (path/host routing + TLS). Use LoadBalancer for non-HTTP (TCP/UDP) or quick QA exposure.

Interview line: “We use an Ingress Controller (single LB) for production routing of many microservices; LoadBalancer is used for TCP/DB or quick dev QA.”

5) Ingress Controller & testing in local labs

Keywords: NGINX Ingress, rules (host/path), kubectl port-forward or NodePort for local testing, curl -H "Host: host" http://localhost:PORT.

Why: Single entry point for multiple services — path-based routing: /auth, /payments, /reports.

Interview line: “Ingress routes host/path to services; locally we test via port-forward or NodePort and send Host header.”

6) Pod scheduling & distribution basics

Keywords: kube-scheduler decides placement (resources, affinity/anti-affinity, taints/tolerations). Default scheduler spreads pods for availability.

Why: Prevents all replicas failing on one node; can be overridden with nodeSelector/nodeAffinity.

Interview line: “Scheduler balances pods across nodes by default; use affinity/anti-affinity to force placement.”

7) Probes: readiness, liveness, startup

Keywords: readiness (serve traffic), liveness (restart unhealthy), startup (give more start time).

Why: Prevents sending traffic to unready pods and ensures recovery/restarts when needed.

Interview line: “Use readiness for traffic gating and liveness/startup to control restart behavior and start timeouts.”

8) ConfigMap & Secret — config management

Keywords: ConfigMap = non-sensitive; Secret = sensitive (base64); mount as env or files. Encryption at rest needed for production.

Why: Externalize config, rotate secrets without code change, maintain 12-factor separation.

Interview line: “We keep DB host/feature flags in ConfigMaps and DB creds in Secrets — in prod we integrate with AWS Secrets Manager or Vault.”

Commands: kubectl create configmap, kubectl create secret generic, valueFrom: secretKeyRef in manifests

9) RBAC, ServiceAccount & CI/CD deployment flow

Keywords: ServiceAccount (robot user), Role/ClusterRole + RoleBinding, Jenkins/Argo/CI uses a service account token, no admin/root for daily deploys.

Why: Least privilege, auditable pipelines, change requests govern production deploys.

Interview line: “Deployments are executed by CI/CD pipelines using service accounts with limited RBAC; admin accounts are reserved for bootstrap/break-glass.”

Process: CR → Git merge → CI pipeline (jenkins-sa) → kubectl apply (namespaced RBAC) → health checks

10) Scaling: manual, HPA, VPA, Cluster Autoscaler

Keywords: Manual scale (kubectl scale), HPA (Horizontal) based on CPU/memory/custom metrics, VPA adjusts resource requests, Cluster Autoscaler adjusts nodes.

Why: Handle variable load and avoid resource starvation.

Interview line: “We scale microservices with HPA for pods and Cluster Autoscaler for node capacity; VPA can tune requests when needed.”

11) Troubleshooting & logs (practical sequence)

Keywords: kubectl get pods, kubectl describe pod, kubectl logs, kubectl get events, stern or kubectl logs -l. Look for OOMKilled, ImagePullBackOff, CrashLoopBackOff.

Why: Find who/why/when pods die — events show controller/kubelet actions; describe shows lastState; logs show app errors.

Interview line: “I check events + pod describe first to find cause (OOM, eviction), then view logs or use stern for multi-pod logs.”

Commands:

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl describe pod <pod>

kubectl logs -l app=myapp --all-containers=true or stern myapp

12) Networking internals & debugging

Keywords: CNI (Calico/Flannel/Cilium), kube-proxy (iptables/IPVS), flat cluster network (pod IPs), CoreDNS for service discovery.

Why: Pod-to-pod connectivity, DNS, and service routing depend on CNI and kube-proxy.

Interview line: “Pods get IPs in a flat network via CNI; services are virtual IPs handled by kube-proxy and CoreDNS provides name resolution.”

Debugging: kubectl exec -it pod -- nslookup service, curl service:port, check CNI/plugin pods

13) Persistent storage & stateful apps (concept)

Keywords: emptyDir, hostPath, PV (PersistentVolume), PVC (PersistentVolumeClaim), StatefulSet for stable IDs.

Why: Databases and stateful services need persistent storage and stable identity.

Interview line: “Databases use PV/PVC (e.g., EBS on AWS) and StatefulSet for stable pod identity and ordered lifecycle.”

(We planned hands-on later.)

14) Observability & monitoring (real-world)

Keywords: Prometheus + Grafana, CloudWatch, alerts, dashboards, log aggregation (ELK/Fluentd), Slack alerts.

Why: Proactive monitoring and alerting for production health and rollback triggers.

Interview line: “We used Prometheus/Grafana for metrics, CloudWatch for infra, and Slack alerts for critical thresholds (latency, error rates).”

15) KillerKoda / local lab notes (practical)

Keywords: controlplane VM vs worker nodes, kubectl get pods shows default namespace only, -n <ns> or -A shows all.

Why: Local labs map LoadBalancer to NodePort or pending EXTERNAL-IP — test via NodePort or kubectl port-forward.

Test tips:

kubectl get svc -n ingress-nginx to find NodePort (e.g., 80:31556/TCP)

curl -H "Host: bank.com" http://localhost:31556 or curl -H "Host: bank.com" http://127.0.0.1:NodePort

Port-forward ingress controller: kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80 then curl -H "Host: bank.com" http://localhost:8080

16) Manifests & deployment practice patterns

Keywords: Separate YAMLs (deployment, svc, ingress, configmap, secret) in a k8s-manifests/ folder; kubectl apply -f folder/.

Why: Maintainability + CI/CD. Helm or GitOps (ArgoCD/Flux) for production.

Interview line: “In prod we manage manifests via Helm/GitOps; in CI we helm upgrade or kubectl apply -f manifests/ for automation.”

17) Common interview Qs we prepared for (quick list)

Why do we need a Service?

Differences: Pod vs Deployment vs StatefulSet vs DaemonSet.

How does K8s self-heal a crashed pod? (controller notices desired != actual → recreate → scheduler → kubelet runs)

NodePort vs LoadBalancer vs Ingress — when use which?

How do you store/rotate secrets? (K8s Secret vs Vault/Secrets Manager)

How to troubleshoot CrashLoopBackOff / OOMKilled? (describe + logs + events)

Who runs deployments in production? (CI/CD service accounts + RBAC + change requests)

18) Quick command cheat (copy/paste)
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods -A
kubectl get svc
kubectl apply -f <file-or-folder>
kubectl describe pod <pod>
kubectl logs <pod> [-c container]
kubectl logs -l app=myapp --all-containers=true
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx 8080:80
curl -H "Host: bank.com" http://localhost:8080
kubectl create configmap app-config --from-literal=KEY=VALUE
kubectl create secret generic db-secret --from-literal=DB_PASS=secret
kubectl scale deployment myapp --replicas=5
kubectl autoscale deployment myapp --cpu-percent=50 --min=2 --max=10

(interviews)

“We ran 10 microservices for the Client Portal on EKS; Ingress + single LB routed traffic to microservices (paths/hosts), Secrets were stored via Vault/AWS Secrets Manager and injected into pods, deployments were automated by Jenkins pipelines running under service accounts with RBAC, and we used Prometheus/Grafana for monitoring and HPA + Cluster Autoscaler for scaling.”

END
