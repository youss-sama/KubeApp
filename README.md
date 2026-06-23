# Déploiement Kubernetes d'une application web 3-tiers
 
[🇫🇷 Français](#français) | [🇬🇧 English](#english)
 
---
 
## Français
 
### 📋 Description
 
Projet personnel de déploiement d'une application web 3-tiers sur un cluster Kubernetes auto-hébergé (1 nœud master + 1 nœud worker, installé manuellement avec kubeadm).
 
L'application est composée de :
- **Frontend** : Nginx
- **Backend** : API REST en Flask (Python)
- **Base de données** : PostgreSQL
L'objectif de ce projet était de mettre en pratique les concepts fondamentaux de Kubernetes : isolation des ressources, gestion de la configuration et des secrets, persistance des données, exposition des services et mise à l'échelle automatique.
 
### 🏗️ Architecture
 
```
                        ┌─────────────────┐
        Internet ──────▶│  Ingress (nginx) │
                        └────────┬─────────┘
                                 │
                  ┌──────────────┴──────────────┐
                  │                              │
          ┌───────▼────────┐            ┌────────▼────────┐
          │ Frontend (nginx)│            │ Backend (Flask) │
          │  2 replicas     │            │  2-6 replicas   │
          │                 │            │  (HPA: CPU 50%) │
          └─────────────────┘            └────────┬────────┘
                                                   │
                                          ┌────────▼────────┐
                                          │  PostgreSQL     │
                                          │  + PVC (1Gi)    │
                                          └─────────────────┘
```
 
### 🧩 Ressources Kubernetes utilisées
 
| Ressource | Rôle |
|---|---|
| Namespace | Isolation des ressources du projet |
| Deployment | Gestion des pods (frontend, backend, base de données) |
| Service (ClusterIP) | Communication interne entre les composants |
| ConfigMap | Configuration non sensible et injection du code applicatif |
| Secret | Identifiants de connexion PostgreSQL |
| PersistentVolumeClaim | Persistance des données PostgreSQL |
| Ingress | Exposition HTTP du frontend et du backend |
| HorizontalPodAutoscaler | Scaling automatique du backend (2 à 6 pods, seuil CPU 50%) |
 
### 🛠️ Stack technique
 
Kubernetes · kubectl · Nginx · Ingress Controller (nginx) · PostgreSQL · Flask · metrics-server · local-path-provisioner · YAML · Linux
 
### 📁 Structure du dépôt
 
```
k8s-3tier-app/
├── 00-namespace.yaml
├── db/
│   ├── secret.yaml
│   ├── pvc.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── backend/
│   ├── configmap.yaml
│   ├── app-code-configmap.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
├── frontend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── README.md
```
 
### ⚙️ Prérequis
 
- Cluster Kubernetes opérationnel (kubeadm)
- `kubectl` configuré
- [metrics-server](https://github.com/kubernetes-sigs/metrics-server)
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/)
- [local-path-provisioner](https://github.com/rancher/local-path-provisioner) (StorageClass pour environnement bare-metal)
### 🚀 Installation
 
**1. Installer les composants du cluster**
 
```bash
# metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
 
# ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
 
# local-path-provisioner
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
 
**2. Déployer l'application**
 
```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f db/
kubectl apply -f backend/
kubectl apply -f frontend/
```
 
**3. Configurer l'accès**
 
```bash
kubectl get nodes -o wide          # récupérer l'IP d'un node
kubectl get svc -n ingress-nginx   # récupérer le NodePort (ex: 32057)
```
 
Ajouter dans `/etc/hosts` :
```
<IP_NODE>  webapp.local
```
 
Accès : `http://webapp.local:<PORT>`
 
### ✅ Vérification
 
```bash
kubectl get all -n webapp
kubectl get ingress -n webapp
kubectl get hpa -n webapp
```
 
Endpoints disponibles :
- `GET /` → frontend (nginx)
- `GET /api` → statut du backend
- `GET /api/health` → health check
- `GET /api/items` → liste de données factices
### 📈 Test de l'autoscaling
 
```bash
kubectl run load-test -n webapp --image=busybox --restart=Never -it --rm \
  -- /bin/sh -c "while true; do wget -q -O- backend-service.webapp.svc.cluster.local:5000; done"
```
 
```bash
watch kubectl get hpa -n webapp
```
 
### ⚠️ Limites connues
 
- Backend non packagé dans une image Docker dédiée (code injecté via ConfigMap dans une image Python générique)
- Secrets encodés en base64, sans chiffrement réel (Vault/Sealed Secrets recommandés en production)
- Absence de liveness/readiness probes
- Pas de pipeline CI/CD
### 🔭 Pistes d'amélioration
 
- Build d'une image Docker dédiée pour le backend
- Ajout de liveness/readiness probes
- Chiffrement des secrets (Sealed Secrets, Vault)
- Pipeline CI/CD (GitHub Actions)
- Gestion multi-environnement (Helm/Kustomize)
- Monitoring (Prometheus + Grafana)
---
 
## English
 
### 📋 Description
 
Personal project deploying a 3-tier web application on a self-hosted Kubernetes cluster (1 master node + 1 worker node, manually installed with kubeadm).
 
The application consists of:
- **Frontend**: Nginx
- **Backend**: REST API in Flask (Python)
- **Database**: PostgreSQL
The goal of this project was to practice core Kubernetes concepts: resource isolation, configuration and secrets management, data persistence, service exposure, and automatic scaling.
 
### 🏗️ Architecture
 
```
                        ┌─────────────────┐
        Internet ──────▶│  Ingress (nginx) │
                        └────────┬─────────┘
                                 │
                  ┌──────────────┴──────────────┐
                  │                              │
          ┌───────▼────────┐            ┌────────▼────────┐
          │ Frontend (nginx)│            │ Backend (Flask) │
          │  2 replicas     │            │  2-6 replicas   │
          │                 │            │  (HPA: CPU 50%) │
          └─────────────────┘            └────────┬────────┘
                                                   │
                                          ┌────────▼────────┐
                                          │  PostgreSQL     │
                                          │  + PVC (1Gi)    │
                                          └─────────────────┘
```
 
### 🧩 Kubernetes resources used
 
| Resource | Role |
|---|---|
| Namespace | Isolates the project's resources |
| Deployment | Manages pods (frontend, backend, database) |
| Service (ClusterIP) | Internal communication between components |
| ConfigMap | Non-sensitive configuration and app code injection |
| Secret | PostgreSQL credentials |
| PersistentVolumeClaim | PostgreSQL data persistence |
| Ingress | HTTP exposure of frontend and backend |
| HorizontalPodAutoscaler | Automatic backend scaling (2 to 6 pods, 50% CPU threshold) |
 
### 🛠️ Tech stack
 
Kubernetes · kubectl · Nginx · Ingress Controller (nginx) · PostgreSQL · Flask · metrics-server · local-path-provisioner · YAML · Linux
 
### 📁 Repository structure
 
```
k8s-3tier-app/
├── 00-namespace.yaml
├── db/
│   ├── secret.yaml
│   ├── pvc.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── backend/
│   ├── configmap.yaml
│   ├── app-code-configmap.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── hpa.yaml
├── frontend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── README.md
```
 
### ⚙️ Prerequisites
 
- A running Kubernetes cluster (kubeadm)
- `kubectl` configured
- [metrics-server](https://github.com/kubernetes-sigs/metrics-server)
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/)
- [local-path-provisioner](https://github.com/rancher/local-path-provisioner) (StorageClass for bare-metal environments)
### 🚀 Installation
 
**1. Install cluster components**
 
```bash
# metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
 
# ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
 
# local-path-provisioner
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
 
**2. Deploy the application**
 
```bash
kubectl apply -f 00-namespace.yaml
kubectl apply -f db/
kubectl apply -f backend/
kubectl apply -f frontend/
```
 
**3. Configure access**
 
```bash
kubectl get nodes -o wide          # get a node's IP
kubectl get svc -n ingress-nginx   # get the NodePort (e.g. 32057)
```
 
Add to `/etc/hosts`:
```
<NODE_IP>  webapp.local
```
 
Access: `http://webapp.local:<PORT>`
 
### ✅ Verification
 
```bash
kubectl get all -n webapp
kubectl get ingress -n webapp
kubectl get hpa -n webapp
```
 
Available endpoints:
- `GET /` → frontend (nginx)
- `GET /api` → backend status
- `GET /api/health` → health check
- `GET /api/items` → sample data list
### 📈 Testing autoscaling
 
```bash
kubectl run load-test -n webapp --image=busybox --restart=Never -it --rm \
  -- /bin/sh -c "while true; do wget -q -O- backend-service.webapp.svc.cluster.local:5000; done"
```
 
```bash
watch kubectl get hpa -n webapp
```
 
### ⚠️ Known limitations
 
- Backend not packaged into a dedicated Docker image (code injected via ConfigMap into a generic Python image)
- Secrets base64-encoded, not truly encrypted (Vault/Sealed Secrets recommended in production)
- No liveness/readiness probes
- No CI/CD pipeline
### 🔭 Possible improvements
 
- Build a dedicated Docker image for the backend
- Add liveness/readiness probes
- Encrypt secrets (Sealed Secrets, Vault)
- CI/CD pipeline (GitHub Actions)
- Multi-environment management (Helm/Kustomize)
- Monitoring (Prometheus + Grafana)
---
 
## 👤 Author
 
**Younes HAJJI**
