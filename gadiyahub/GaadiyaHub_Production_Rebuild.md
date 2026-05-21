# GaadiyaHub — Production Rebuild Document
**Owner:** Digamber Dwivedi  
**Started:** May 2026  
**Goal:** Enterprise grade production platform on AWS Production Account

---

## OBJECTIVE

- Do not touch the root account (mixed/manual setup) — client production is running there.
- Build a clean, enterprise grade, everything-as-code platform on the Production AWS account.
- Deliver a modern, optimized, production-ready platform as a surprise to the client.

---

## ARCHITECTURE OVERVIEW

```
AWS Production Account (922981236957)
│
├── VPC (10.2.0.0/16) — Terraform
│   ├── Public Subnet 1 (10.2.1.0/24) — ap-south-1a
│   ├── Public Subnet 2 (10.2.2.0/24) — ap-south-1b
│   └── Private Subnet 1 (10.2.3.0/24) — ap-south-1a
│
├── EC2 — Control Plane (t3.large) — 13.234.218.197
├── EC2 — Worker 1 (t3.large) — 3.7.148.96
├── EC2 — Worker 2 (t3.medium)
├── ALB — gadiyahub-alb
├── RDS — MSSQL (to be migrated)
└── S3 — Terraform state + backups
```

---

## PHASE TRACKER

### ✅ Phase 1 — Infrastructure (DONE)

| Task | Status | Notes |
|------|--------|-------|
| AWS Organizations setup | ✅ Done | Production + Staging + Root accounts |
| S3 Terraform state backend | ✅ Done | gadiyahub-tf-state bucket |
| Terraform — VPC, Subnets, IGW | ✅ Done | 10.2.0.0/16 |
| Terraform — Security Groups | ✅ Done | K8s ports open |
| Terraform — EC2 x3 | ✅ Done | Control plane + 2 workers |
| Terraform — ALB + Target Group | ✅ Done | |
| Terraform — Elastic IPs | ✅ Done | |
| Kubeadm — Control plane init | ✅ Done | K8s v1.31.14 |
| Flannel CNI | ✅ Done | |
| Ansible — Worker node setup | ✅ Done | Playbook: k8s-workers.yml |
| Ansible — Workers join cluster | ✅ Done | Playbook: join-workers.yml |
| 3 Node cluster Ready | ✅ Done | kubectl get nodes = 3 Ready |

---

### 🔄 Phase 2 — Platform Setup (IN PROGRESS)

| Task | Status | Notes |
|------|--------|-------|
| Helm install | ⏳ Pending | |
| Nginx Ingress Controller | ⏳ Pending | |
| cert-manager SSL | ⏳ Pending | Let's Encrypt |
| Prometheus + Grafana | ⏳ Pending | kube-prometheus-stack |
| PostgreSQL | ⏳ Pending | Helm chart |
| Redis | ⏳ Pending | Helm chart |
| RabbitMQ | ⏳ Pending | Helm chart |
| OpenSearch (ECK) | ⏳ Pending | |

---

### ⏳ Phase 3 — Application Migration

| Task | Status | Notes |
|------|--------|-------|
| account service | ⏳ Pending | |
| content service | ⏳ Pending | |
| dealer service | ⏳ Pending | |
| newvehicle service | ⏳ Pending | |
| usedvehicle service | ⏳ Pending | |
| website (Next.js frontend) | ⏳ Pending | |
| engagement service | ⏳ Pending | |
| blog service | ⏳ Pending | |

---

### ⏳ Phase 4 — CI/CD

| Task | Status | Notes |
|------|--------|-------|
| GitHub Actions — staging pipelines | ⏳ Pending | |
| Jenkins — production pipeline | ⏳ Pending | With approval gate |
| Self-hosted runner | ⏳ Pending | On control plane EC2 |

---

### ⏳ Phase 5 — Enterprise Features

| Task | Status | Notes |
|------|--------|-------|
| Lambda — media handler | ⏳ Pending | Replicate from root account |
| DR setup | ⏳ Pending | RTO 5hr, RPO 12hr |
| Backup automation | ⏳ Pending | PostgreSQL → S3 |
| AWS Cost alerts | ⏳ Pending | |
| DNS cutover | ⏳ Pending | gadiyahub.com → new infra |

---

## THEORY CONCEPTS — QUICK REFERENCE

### Helm
```
Helm is a package manager for Kubernetes.

Chart   = A package (similar to .deb in Ubuntu)
Release = A deployed instance of a chart
Repo    = A collection of charts (like apt repository)

Key Commands:
helm install <release-name> <chart>     — Install a chart
helm upgrade <release-name> <chart>     — Upgrade a release
helm rollback <release-name> <revision> — Rollback to previous version
helm list                               — List all releases
helm history <release-name>             — Show release history
helm repo add <name> <url>             — Add a chart repository
helm repo update                        — Update local repo cache
```

### Kubernetes Key Concepts
```
Pod         = Smallest deployable unit. Contains one or more containers.
Deployment  = Manages a set of pods. Provides self-healing and rolling updates.
Service     = Stable network endpoint to access pods.
Ingress     = HTTP/HTTPS routing rules into the cluster.
ConfigMap   = Non-sensitive configuration data.
Secret      = Sensitive data stored as base64 (passwords, tokens).
PVC         = Persistent Volume Claim — requests storage for a pod.
Namespace   = Logical isolation between workloads.
DaemonSet   = Runs one pod per node (e.g. node-exporter).
StatefulSet = For stateful apps like databases (stable pod names).
```

### Ansible
```
Inventory = List of target machines (IPs or hostnames)
Playbook  = YAML file describing what to do
Module    = A single unit of work (apt, service, copy, shell)
Role      = Reusable, structured collection of tasks
Handler   = Task that runs only when notified by another task

Playbook Structure:
---
- name: Description of the play
  hosts: workers
  become: yes        # Run as sudo

  tasks:
    - name: Description of the task
      module_name:
        parameter: value
        parameter: value
```

### Terraform
```
provider  = Declares the cloud provider and credentials
resource  = Defines an infrastructure component
variable  = Input value to make code reusable
output    = Value returned after apply
module    = Reusable group of resources

Key Commands:
terraform init     = Initialize working directory, download providers
terraform plan     = Preview what will be created/changed/destroyed
terraform apply    = Create or update real infrastructure
terraform destroy  = Tear down all managed resources
terraform show     = Display current state
```

### Jenkins Declarative Pipeline
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t app:latest .'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl set image deployment/app app=app:latest'
            }
        }
    }
}
```

### AWS Lambda
```
Lambda is a serverless compute service.
Code runs in response to events — no server management required.
Billing is per execution (number of requests + duration).

Key Concepts:
Function        = The code that runs
Trigger         = The event that invokes the function
Execution Role  = IAM role granting permissions to the function
Layer           = Shared libraries/dependencies across functions

Common Triggers:
- API Gateway   (HTTP requests)
- S3            (file upload/delete events)
- EventBridge   (scheduled runs)
- SQS           (message queue)
```

### 6R Migration Strategy
```
Rehost      = Lift and Shift. Move as-is to cloud. No changes.
Replatform  = Move and optimize. Minor changes (e.g. MySQL → RDS).
Refactor    = Re-architect. Redesign for cloud-native (e.g. monolith → microservices).
Repurchase  = Replace with SaaS product (e.g. CRM → Salesforce).
Retire      = Decommission. Application is no longer needed.
Retain       = Keep on-premise for now. Too complex or compliance issues.
```

### AWS Well-Architected Framework — 6 Pillars
```
Security               = Protect data and systems. IAM, encryption, VPC.
Operational Excellence = Run and monitor systems. Automation, runbooks, CI/CD.
Reliability            = Recover from failures. Multi-AZ, backups, Auto Scaling.
Performance Efficiency = Use resources efficiently. Right sizing, caching, CDN.
Cost Optimization      = Avoid unnecessary spend. Reserved instances, right sizing.
Sustainability         = Reduce environmental impact. Efficient resource use.

Memory aid: SORPES
```

### Docker Key Concepts
```
Image      = Read-only template to create containers
Container  = Running instance of an image
Dockerfile = Instructions to build an image
Registry   = Storage for images (DockerHub, ECR)
Layer      = Each instruction in Dockerfile creates a layer

Key Commands:
docker build -t name:tag .     — Build image from Dockerfile
docker run -d -p 80:80 name    — Run container in background
docker ps                      — List running containers
docker logs <container>        — View container logs
docker exec -it <container> sh — Shell into container
docker push name:tag           — Push image to registry
```

---

## INCIDENTS & LEARNINGS

| Date | Issue | Fix | Learning |
|------|-------|-----|----------|
| May 2026 | kubectl timeout after location change | Updated SG IP from 152.58 to 152.59 | Always check current IP when changing location |
| May 2026 | Workers could not join cluster (timeout on port 6443) | Added inbound rule for 10.2.0.0/16 on port 6443 | Internal VPC traffic also needs explicit SG rules |
| May 2026 | Ansible ping timeout to workers | Added inbound rule for 10.2.0.0/16 on port 22 | Control plane needs SSH access to workers internally |
| May 2026 | Terraform state drift on .gitignore | Resolved merge conflict — merged both versions | Always pull before push; .gitignore should be in root |

---

## IMPORTANT IPs & ENDPOINTS

```
Control Plane Public IP : 13.234.218.197
Worker 1 Public IP      : 3.7.148.96
ALB DNS                 : gadiyahub-alb-974612399.ap-south-1.elb.amazonaws.com

Control Plane Private IP : 10.2.1.23
Worker 1 Private IP      : 10.2.1.43
Worker 2 Private IP      : 10.2.2.181

SSH to Control Plane:
ssh -i ~/.ssh/gadiyahub-prod-key.pem ubuntu@13.234.218.197

Check current home IP:
curl -s https://checkip.amazonaws.com
```

---

## USEFUL COMMANDS

```bash
# ── Cluster Health ──────────────────────────────────────
kubectl get nodes
kubectl get pods --all-namespaces
kubectl get pods -n <namespace>

# ── Helm ────────────────────────────────────────────────
helm list -A
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install <name> <chart> --namespace <ns>
helm upgrade <name> <chart> --namespace <ns>
helm rollback <name> <revision>
helm history <name>

# ── Ansible ─────────────────────────────────────────────
cd ~/ansible-k8s
ansible -i inventory.ini workers -m ping
ansible-playbook -i inventory.ini k8s-workers.yml
ansible-playbook -i inventory.ini join-workers.yml

# ── Terraform ───────────────────────────────────────────
cd ~/workspace/terraform-practice/production
aws sso login --profile production-gadiyahub
terraform plan
terraform apply
terraform destroy

# ── SSH ─────────────────────────────────────────────────
ssh -i ~/.ssh/gadiyahub-prod-key.pem ubuntu@13.234.218.197
ssh -i ~/.ssh/gadiyahub-prod-key.pem ubuntu@3.7.148.96
```

---

## SESSION LOG

| Date | What Was Done |
|------|---------------|
| May 18, 2026 | Terraform apply — 17 resources provisioned in Production account |
| May 18, 2026 | Kubeadm — control plane initialized, Flannel CNI deployed |
| May 18, 2026 | Ansible — worker nodes configured (containerd, kubeadm, kubelet) |
| May 18, 2026 | Ansible — workers joined cluster. 3 nodes Ready. |
| May 21, 2026 | Machines restarted after cost-saving stop. IP updated in Security Group. |
| May 21, 2026 | Helm install — next step |

---

*Update this document at the end of every session.*
