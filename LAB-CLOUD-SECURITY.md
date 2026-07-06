LAB: Enterprise Cloud Security & DevSecOps Platform
🎯 Objetivo do laboratório

Simular um ambiente corporativo real com:

Multi-cloud (AWS + Azure)
Kubernetes seguro (AKS / EKS-ready)
Pipeline DevSecOps completo
Segurança em runtime + supply chain
Compliance contínuo (CIS, NIST)
Observabilidade + SIEM-like
Resposta a incidentes automatizada

🧠 1. Arquitetura de Alto Nível
                ┌──────────────────────────────┐
                │ GitHub / GitLab (Source)     │
                └────────────┬─────────────────┘
                             │
                             ▼
        ┌────────────────────────────────────────┐
        │ DevSecOps CI/CD Pipeline              │
        │ tfsec | checkov | trivy | opa | tflint│
        └────────────┬──────────────────────────┘
                     │
                     ▼
     ┌──────────────────────────────────────────┐
     │ Terraform IaC Layer                      │
     │ AWS + Azure + Kubernetes                │
     └────────────┬───────────────────────────┘
                  │
      ┌───────────┼──────────────────────────────┐
      ▼           ▼                              ▼
┌─────────┐  ┌────────────┐             ┌──────────────┐
│ AWS     │  │ Azure      │             │ Kubernetes   │
│ GuardDuty│  │ Defender   │             │ (AKS/EKS)    │
│ Security │  │ Policy     │             │ Kyverno/OPA  │
│ Hub      │  │ KeyVault   │             │ NetworkPolicy│
└────┬────┘  └────┬───────┘             └──────┬───────┘
     │            │                            │
     ▼            ▼                            ▼
┌────────────────────────────────────────────────────┐
│ Observability & Security Intelligence              │
│ Prometheus | Grafana | Loki | Falco | OpenSearch  │
└────────────────────────────────────────────────────┘

🏗️ 2. Estrutura do Repositório (Profissional)
cloud-security-devsecops-lab/

├── 00-architecture/
│   └── high-level-diagram.png
│
├── 01-terraform/
│   ├── modules/
│   │   ├── aws-security/
│   │   ├── azure-security/
│   │   ├── eks-cluster/
│   │   ├── aks-cluster/
│   │   ├── network-hardening/
│   │   └── logging/
│   │
│   ├── envs/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│
├── 02-kubernetes-security/
│   ├── rbac/
│   ├── network-policies/
│   ├── kyverno-policies/
│   ├── opa-gatekeeper/
│   └── pod-security-standards/
│
├── 03-container-security/
│   ├── secure-dockerfile/
│   ├── distroless-images/
│   ├── trivy-scans/
│   └── sbom/
│
├── 04-devsecops-pipeline/
│   ├── github-actions/
│   ├── gitlab-ci/
│   ├── terraform-pipeline.yml
│   └── security-gates.yml
│
├── 05-monitoring-siem/
│   ├── prometheus/
│   ├── grafana/
│   ├── falco-runtime/
│   └── opensearch/
│
├── 06-compliance/
│   ├── cis-aws-benchmark/
│   ├── cis-kubernetes/
│   ├── nist-controls/
│   └── policy-as-code/
│
└── README.md

☁️ 3. AWS Security (Cloud Security Engineer)
🔐 IAM Least Privilege + MFA Enforcement
resource "aws_iam_account_password_policy" "strict" {
  minimum_password_length        = 14
  require_lowercase_characters   = true
  require_numbers                = true
  require_uppercase_characters   = true
  require_symbols                = true
  max_password_age              = 90
  hard_expiry                   = true
}
🛡️ GuardDuty + Threat Detection
resource "aws_guardduty_detector" "main" {
  enable = true
}
🔍 CloudTrail + Auditability
resource "aws_cloudtrail" "audit" {
  name                          = "org-trail"
  is_multi_region_trail         = true
  include_global_service_events = true
  enable_logging                = true

  enable_log_file_validation = true
}
🔑 KMS Encryption Everywhere
resource "aws_kms_key" "security" {
  description             = "Enterprise Security Key"
  enable_key_rotation     = true
  deletion_window_in_days = 10
}

☁️ 4. Azure Security (Cloud Security Architect)
🛡️ Defender for Cloud
resource "azurerm_security_center_subscription_pricing" "containers" {
  tier          = "Standard"
  resource_type = "Containers"
}
🔐 Key Vault Hardened
resource "azurerm_key_vault" "secure" {
  tenant_id                  = var.tenant_id
  sku_name                   = "standard"
  soft_delete_retention_days = 90
  purge_protection_enabled   = true
}
🚫 Azure Policy (Zero Trust)
resource "azurerm_policy_assignment" "deny_public_ip" {
  name                 = "deny-public-ip"
  policy_definition_id = data.azurerm_policy_definition.no_public_ip.id
}

☸️ 5. Kubernetes Security (Cloud Security Engineer + DevSecOps)
🚫 Default Deny Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

🔐 Pod Security Standard (Baseline Hardening)
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: nginx
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
🔐 RBAC mínimo privilégio
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

🧠 OPA / Kyverno Policy (Compliance as Code)
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  validationFailureAction: enforce
  rules:
  - name: check-root
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Containers must not run as root"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true

🐳 6. Docker Security (Supply Chain Hardening)
🔒 Secure Dockerfile
FROM gcr.io/distroless/base-debian12

USER 1000:1000

WORKDIR /app

COPY app /app

EXPOSE 8080

CMD ["/app"]

🛡️ docker-compose hardened
services:
  api:
    image: secure-api
    read_only: true
    privileged: false
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    tmpfs:
      - /tmp
🔎 SBOM + Scan
trivy image secure-api
syft secure-api -o spdx-json
grype secure-api
🔁 7. DevSecOps Pipeline (Coração do laboratório)
name: DevSecOps Pipeline

on: [push]

jobs:
  security:
    steps:

    - name: Terraform Validate
      run: terraform validate

    - name: tfsec Scan
      run: tfsec .

    - name: Checkov Scan
      run: checkov -d .

    - name: Trivy Scan
      run: trivy fs .

    - name: Hadolint
      run: hadolint Dockerfile

    - name: OPA Policy Check
      run: opa eval -d policies/ "data"

    - name: Terraform Plan
      run: terraform plan

📊 8. Observability + Security Intelligence (Cybersecurity Engineer)
🔥 Falco Runtime Detection

Detecta:

container shell spawn
privilege escalation
crypto mining behavior
📡 Prometheus + Grafana
CPU anomalies
pod restart spikes
suspicious traffic patterns
🧠 SIEM-like Layer
Logs:
  AWS CloudTrail
  Azure Activity Logs
  Kubernetes Audit Logs
  Container Logs

↓
Loki / OpenSearch

↓
Alerting:
  Slack / Email / Webhook
