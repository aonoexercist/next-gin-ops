# next-gin-ops

Production deployment configuration for a **Next.js + Gin** stack, orchestrated with Kubernetes and proxied through Nginx.

## Stack

| Service      | Image                                           | Role                             |
|--------------|-------------------------------------------------|----------------------------------|
| `frontend`   | `your-dockerhub/nextjs-app`                     | Next.js frontend (port 3000)     |
| `backend`    | `your-dockerhub/gin-api`                        | Gin (Go) REST API (port 8080)    |
| `postgres`   | `postgres:15`                                   | PostgreSQL database (port 5432)  |
| `nginx`      | `nginx:alpine`                                  | Reverse proxy (port 80)          |

## Prerequisites

- A running Kubernetes cluster (local: [minikube](https://minikube.sigs.k8s.io/) / [kind](https://kind.sigs.k8s.io/), or a managed cluster)
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/) configured to target your cluster
- Container images built and pushed to your registry

## Getting Started

1. **Clone the repository**

   ```bash
   git clone https://github.com/aonoexercist/next-gin-ops.git
   cd next-gin-ops
   ```

2. **Configure environment variables**

   Update the `env` values in the manifests (or replace them with `Secret` / `ConfigMap` references) before applying:

   - `k8s-infra/backend/deployment.yaml` — `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`
   - `k8s-infra/db/postgres.yaml` — `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`
   - `k8s-infra/frontend/deployment.yaml` — `NEXT_PUBLIC_API_URL`

3. **Update image references**

   Replace `your-dockerhub/gin-api` and `your-dockerhub/nextjs-app` in the deployment manifests with your actual image paths.

4. **Apply the manifests**

   ```bash
   kubectl apply -f k8s-infra/db/
   kubectl apply -f k8s-infra/backend/
   kubectl apply -f k8s-infra/frontend/
   ```

5. **Access the frontend**

   The frontend service is exposed as a `NodePort` on port `30007`:

   ```bash
   # minikube
   minikube service nextjs-frontend

   # or manually
   kubectl get nodes -o wide   # grab a node IP
   # then open http://<node-ip>:30007
   ```

6. **Tear down**

   ```bash
   kubectl delete -f k8s-infra/
   ```

## Project Structure

```
next-gin-ops/
├── k8s-infra/
│   ├── backend/
│   │   ├── deployment.yaml   # Gin API Deployment
│   │   └── service.yaml      # ClusterIP Service (port 8080)
│   ├── db/
│   │   └── postgres.yaml     # PostgreSQL Deployment, PVC & Service
│   └── frontend/
│       ├── deployment.yaml   # Next.js Deployment
│       └── service.yaml      # NodePort Service (nodePort 30007)
├── nginx/
│   └── default.conf          # Nginx reverse-proxy config
└── README.md
```

## Nginx Routing

`nginx/default.conf` proxies traffic on port 80:

| Path            | Upstream                  |
|-----------------|---------------------------|
| `/`             | `http://frontend:3000`    |
| `/backend-api/` | `http://backend:8080`     |

## Notes

- Credentials in the manifests are plain-text placeholders. For production, use [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).
- The PostgreSQL `PersistentVolumeClaim` requests 1 Gi with `ReadWriteOnce` access mode — adjust to match your storage class.
- The backend pod connects to the database via the `postgres` ClusterIP Service (`postgres:5432`).
