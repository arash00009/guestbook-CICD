# Guestbook Application - Cloud-Native Implementation with ArgoCD

![GitHub](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)
![OpenShift](https://img.shields.io/badge/OpenShift-Kubernetes-red?logo=openshift)
![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-green?logo=argo)
![License](https://img.shields.io/badge/License-MIT-green)

## 📋 Overview

This repository contains a comprehensive GitOps implementation of a distributed guestbook application on OpenShift Kubernetes platform. The project demonstrates modern DevOps practices by integrating Continuous Integration (GitHub Actions) with Continuous Delivery (ArgoCD) to achieve fully automated application deployment.

**Live Application:** https://arash-frontend-grupp3.apps.devops24.cloud2.se/

---

## 🎯 Project Objectives

1. Implement a fully functional guestbook application with distributed architecture (Frontend, Backend, Database, Cache)
2. Establish GitOps-based workflow using ArgoCD for automated and declarative deployment
3. Ensure data persistence via PostgreSQL and caching via Redis
4. Document implementation process including challenges and solutions
5. Demonstrate best practices for security and resource management in shared cluster environments
6. Create reproducible and scalable infrastructure as a model for future projects

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                   │
│                    (OpenShift/grupp3)                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────┐        ┌──────────────────┐     │
│  │   Frontend (2x)  │        │   Backend (2x)   │     │
│  │   Nginx:latest   │        │   Go:latest      │     │
│  └──────────────────┘        └──────────────────┘     │
│                    ▼                ▲                   │
│  ┌──────────────────────────────────────────────┐     │
│  │         PostgreSQL Database (1x)             │     │
│  │     (Persistent Volume Storage)              │     │
│  └──────────────────────────────────────────────┘     │
│  ┌──────────────────────────────────────────────┐     │
│  │         Redis Cache (1x)                     │     │
│  │     (Session & Cache Management)             │     │
│  └──────────────────────────────────────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘
          ▲                              │
          │                              ▼
    ┌─────────────┐            ┌──────────────────┐
    │   GitHub    │            │     ArgoCD       │
    │  (Source of │            │    (GitOps)      │
    │   Truth)    │            │                  │
    └─────────────┘            └──────────────────┘
          ▲                              │
          │                              ▼
    ┌──────────────────────────────────────────┐
    │    GitHub Actions (CI Pipeline)          │
    │    - Build Docker images                 │
    │    - Push to Quay.io                     │
    │    - Run tests                           │
    └──────────────────────────────────────────┘
```

---

## 🛠️ Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Container Orchestration** | OpenShift/Kubernetes | Cloud-native orchestration |
| **GitOps Tool** | ArgoCD | Declarative deployment automation |
| **Frontend** | Nginx | Web server & reverse proxy |
| **Backend** | Go/Golang | Application logic |
| **Database** | PostgreSQL | Persistent data storage |
| **Cache** | Redis | Session & cache management |
| **CI/CD** | GitHub Actions + ArgoCD | Automated pipeline |
| **Container Registry** | Quay.io | Image storage & distribution |
| **Version Control** | Git/GitHub | Source of Truth |

---

## 📁 Repository Structure

```
guestbook-CICD/
├── k8s/
│   ├── arash-frontend.yaml          # Frontend Deployment & Service
│   ├── arash-backend.yaml           # Backend Deployment & Service
│   ├── arash-database.yaml          # PostgreSQL StatefulSet
│   ├── arash-config.yaml            # ConfigMaps & Redis
│   └── arash-secrets.yaml           # Secrets (if applicable)
├── .github/
│   └── workflows/
│       └── ci-cd.yaml               # GitHub Actions CI pipeline
├── README.md                         # This file
└── docker/
    ├── Dockerfile.frontend          # Frontend image build
    └── Dockerfile.backend           # Backend image build
```

---

## 🚀 Quick Start

### Prerequisites

- OpenShift CLI (`oc`) installed
- kubectl installed
- GitHub account with SSH key configured
- Quay.io account (for container images)
- ArgoCD installed in OpenShift cluster

### Deployment Steps

1. **Clone the Repository**
   ```bash
   git clone git@github.com:arash00009/guestbook-CICD.git
   cd guestbook-CICD
   ```

2. **Authenticate with OpenShift**
   ```bash
   oc login --web --server https://api.devops24.cloud2.se:6443
   oc whoami  # Verify login
   ```

3. **Create Kubernetes Resources**
   ```bash
   # Manual deployment (if not using ArgoCD)
   oc apply -f k8s/arash-frontend.yaml
   oc apply -f k8s/arash-backend.yaml
   oc apply -f k8s/arash-database.yaml
   oc apply -f k8s/arash-config.yaml
   ```

4. **Configure ArgoCD (Recommended)**
   - Log in to ArgoCD: https://openshift-gitops-server-openshift-gitops.apps.devops24.cloud2.se/
   - Create new application "arash-argo-guestbook"
   - Set Repository URL: `https://github.com/arash00009/guestbook-CICD.git`
   - Set Path: `k8s`
   - Set Destination: `grupp3` namespace
   - Enable Auto-Sync and Self-Heal

5. **Access Application**
   ```bash
   # Get the frontend URL
   oc get route arash-frontend-service -n grupp3
   ```

---

## 📊 Implementation Details

### Naming Standards

To avoid resource conflicts in the shared cluster, all resources follow strict naming convention:

- **Prefix:** `arash-` (author identifier)
- **Format:** `arash-[component]-[resource-type]`
- **Examples:**
  - `arash-frontend` (Deployment)
  - `arash-backend-service` (Service)
  - `arash-database` (StatefulSet)

### Key Features

#### ✅ Automatic Synchronization
- ArgoCD monitors Git repository for changes
- Automatically applies changes to cluster
- Ensures cluster state matches Git state (GitOps principle)

#### ✅ Self-Healing
- Automatically corrects drift between desired and actual state
- Reverts manual changes to cluster
- Maintains configuration consistency

#### ✅ Resource Pruning
- Automatically removes resources deleted from Git
- Prevents orphaned resources in cluster
- Maintains clean state

#### ✅ High Availability
- Frontend and Backend: 2 replicas each
- Load balancing via Kubernetes Services
- Database: Persistent Volume Storage

---

## 🔐 Security Considerations

### Production vs. School Project

This project was created as an educational exercise. In production environments:

| Aspect | This Project | Production |
|--------|-------------|-----------|
| Repository | Public | Private |
| Secrets | GitHub Secrets | External Secret Management (Vault, etc.) |
| RBAC | Admin-managed | Fine-grained role definitions |
| Authentication | Basic auth | OAuth2/OIDC |
| Image Registry | Public Quay.io | Private registry |

### Best Practices Implemented

- ✅ Secrets stored as encrypted GitHub Secrets
- ✅ Robot Account with minimal write permissions for CI/CD
- ✅ Resource quotas via Kubernetes namespaces
- ✅ Network policies for pod communication
- ✅ RBAC configuration for cluster access control

---

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

The `.github/workflows/ci-cd.yaml` pipeline:

1. **Trigger:** On push to main branch
2. **Build:** Create Docker images for frontend and backend
3. **Test:** Run application tests
4. **Push:** Upload images to Quay.io
5. **Notify:** Trigger ArgoCD synchronization

### Pipeline Flow

```
Code Push (GitHub)
    ↓
GitHub Actions Workflow Triggered
    ↓
Build Docker Images
    ↓
Run Tests & Lint
    ↓
Push to Quay.io Registry
    ↓
ArgoCD Detects Changes
    ↓
Automatic Sync to Kubernetes
    ↓
Application Updated
```

---

## 📝 Challenges & Solutions

### Challenge 1: Resource Naming Conflicts
**Problem:** Shared cluster environment with multiple users
**Solution:** Implemented strict naming standard with unique `arash-` prefix

### Challenge 2: Secure Authentication
**Problem:** Exposing credentials in CI/CD pipeline
**Solution:** Used GitHub Secrets + Quay.io Robot Account with limited permissions

### Challenge 3: YAML Syntax Errors
**Problem:** Indentation errors stopping automation
**Solution:** Proper YAML validation and careful spacing (no tabs)

---

## 📚 Documentation

Full implementation report available:

**Document:** `Arash_Rahimi_Cloud_Native_Guestbook_ArgoCD_OpenShift_2026.pdf`

**Contents:**
- Complete implementation methodology
- Architecture design and decisions
- Step-by-step setup instructions
- Troubleshooting guide
- Security considerations
- Lessons learned and reflections

---

## 👥 Contributors

- **Author:** Arash Rahimi
- **Supervisor:** Jonas Björk
- **Institution:** School Project - DevOps Course

---

## 📖 Learning Outcomes

Through this project, I learned:

- ✅ GitOps principles and ArgoCD implementation
- ✅ Kubernetes resource management in shared environments
- ✅ CI/CD pipeline automation with GitHub Actions
- ✅ Secure credential handling and RBAC
- ✅ Infrastructure as Code (IaC) best practices
- ✅ Importance of proper logging and debugging
- ✅ Stress management and iterative problem solving

---

## 🔧 Useful Commands

```bash
# View ArgoCD Application Status
oc get applications -n openshift-gitops

# Manually Sync ArgoCD Application
argocd app sync arash-argo-guestbook

# View Kubernetes Resources
oc get pods,svc,deployments -n grupp3

# Check Application Logs
oc logs -f deployment/arash-frontend -n grupp3

# Port Forward to Local Machine
oc port-forward svc/arash-frontend-service 8080:80 -n grupp3

# View Kubernetes Events
oc describe pod [pod-name] -n grupp3

# Access ArgoCD CLI
argocd login [argocd-server] --username admin
```

---

## 📞 Support & Feedback

For questions or issues, please:

1. Check the implementation report for detailed documentation
2. Review GitHub Issues section
3. Submit Pull Requests for improvements

---

## 📄 License

This project is provided as-is for educational purposes.

---

## 🔗 References

- [Argo Project](https://argoproj.github.io/)
- [OpenShift Documentation](https://docs.openshift.com/)
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [GitOps Best Practices](https://www.gitops.tech/)

---

**Last Updated:** January 11, 2026  
**Status:** ✅ Active & Maintained

---

## 🎓 Course Information

**Course:** DevOps & Cloud Architecture  
**Assignment:** GitOps Implementation with ArgoCD  
**Date:** Academic Year 2025-2026
