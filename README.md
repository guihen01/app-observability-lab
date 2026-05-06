
# 🚀 Kubernetes Observability Lab

**Python + Prometheus + Grafana + Argo CD + Ingress**

---

## 📌 Overview

Ce lab démontre une chaîne complète **Cloud Native observability + GitOps** :

* Application Python instrumentée (métriques Prometheus)
* Déploiement Kubernetes
* Exposition via Ingress
* Monitoring avec Prometheus
* Visualisation avec Grafana
* Déploiement GitOps avec Argo CD

Based on the lab guide provided in the PDF document : https://github.com/guihen01/app-observability-lab/blob/main/docs/User-Guide.pdf

👉 Objectif : simuler un environnement **production-like** moderne.

---

## 🧠 Architecture

```
User → Ingress → Service → Pods (Python App)
                              ↓
                         /metrics
                              ↓
                        Prometheus
                              ↓
                          Grafana
```

---

## 📁 Project Structure

```
app-observability-lab/
│
├── app-code/
│   ├── app.py
│   └── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── servicemonitor.yaml
│
└── app.yaml   # Argo CD
```

---

## ⚙️ Prerequisites

* Kubernetes cluster (ex: k3d)
* Helm
* kube-prometheus-stack installé
* Docker + Docker Hub
* kubectl

---

## 🐍 Step 1 — Python App (Metrics Enabled)

```python
from flask import Flask
from prometheus_client import Counter, generate_latest

app = Flask(__name__)

REQUESTS = Counter('app_requests_total', 'Total HTTP Requests')

@app.route("/")
def home():
    REQUESTS.inc()
    return "Hello from Observability Lab 🚀"

@app.route("/metrics")
def metrics():
    return generate_latest(), 200, {'Content-Type': 'text/plain'}
```

👉 Endpoint clé :

```
/metrics
```

---

## 🐳 Step 2 — Docker

```bash
docker build -t <dockerhub>/python-app:1.0 .
docker push <dockerhub>/python-app:1.0
```

---

## ☸️ Step 3 — Kubernetes Deployment

### Deployment

* 2 replicas
* expose port 5000

### Service

⚠️ IMPORTANT :

* label `app: python-app`
* port nommé `http`

### Ingress

```
http://python.local
```

---

## 🔥 Step 4 — ServiceMonitor (CRITICAL)

```yaml
kind: ServiceMonitor
```

👉 Permet à Prometheus de scraper :

```
/metrics
```

⚠️ Point critique :
Le label doit matcher Helm :

```yaml
labels:
  release: kube-prometheus-stack
```

---

## 🔁 Step 5 — GitOps avec Argo CD

```yaml
kind: Application
```

* Source : GitHub repo
* Sync automatique
* Self-healing activé

```bash
kubectl apply -f app.yaml
```

---

## ✅ Step 6 — Verification

### Pods

```bash
kubectl get pods
```

👉 attendu :

```
python-app-xxx   Running
```

---

### Application

```bash
kubectl port-forward svc/python-app 8081:80
```

👉 [http://localhost:8081](http://localhost:8081)

---

### Metrics

👉 [http://localhost:8081/metrics](http://localhost:8081/metrics)

---

## 📊 Step 7 — Prometheus

```bash
kubectl port-forward svc/<prometheus-service> -n monitoring 9090
```

👉 [http://localhost:9090](http://localhost:9090)

### Query :

```
app_requests_total
rate(app_requests_total[1m])
```

---

## 📈 Step 8 — Grafana

```bash
kubectl port-forward svc/<grafana-service> -n monitoring 3000:80
```

👉 [http://localhost:3000](http://localhost:3000)

Login :

```
user: admin
password: kubectl get secret ...
```

👉 Visualisation :

* Pods
* CPU / Memory
* métriques custom app

---

## 🔄 Step 9 — GitOps Test

Modifier :

```yaml
replicas: 2 → 4
```

```bash
git commit -m "scale app"
git push
```

👉 Résultat :

* Argo CD détecte le changement
* Scaling automatique Kubernetes

---

## 🎯 Key Learnings

* 🔹 Instrumentation applicative (Prometheus client)
* 🔹 Différence Service vs ServiceMonitor
* 🔹 Importance des labels Kubernetes
* 🔹 GitOps workflow avec Argo CD
* 🔹 Observabilité full stack

---

## ⚠️ Common Pitfalls

* ❌ mauvais label dans ServiceMonitor
* ❌ port non nommé dans Service
* ❌ image Docker non pushée
* ❌ Prometheus non installé

---

## 🧪 Bonus Improvements

* Ajouter dashboards Grafana custom
* Ajouter métriques CPU/memory par app
* Ajouter alerting (AlertManager)
* Version Helm de l’app

---

## 📚 References

* Prometheus Helm Charts
* Kubernetes Docs
* Argo CD Docs

---

## 💡 Why this lab matters

Ce lab démontre des compétences clés recherchées :

* Kubernetes production workflows
* Observability moderne
* GitOps (très demandé)
* Cloud-native design
