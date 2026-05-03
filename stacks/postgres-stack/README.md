# Postgres Stack - Kubernetes-First PostgreSQL Environment

This stack deploys PostgreSQL with pgAdmin for local development and operations workflows.

![Postgres Architecture Diagram](./assets/postgres-stack.drawio.svg)

## Cluster Setup

Set up the base k3d/k3s cluster first using [the main cluster setup guide](../../INSTALL.md).

For the full repository overview, see [the root README](../../README.md).

## Deployment Priority

1. Kubernetes via Argo CD + Kustomize (primary)

## Stack Contents

- `k8s/`: Kubernetes manifests for Postgres + pgAdmin
- `argocd/postgres-stack-app.yaml`: Argo CD Application manifest
- `docker-compose.yml`: Legacy Docker Compose deployment (unmaintained)

## Kubernetes Deployment (Primary)

### Prerequisites

- Kubernetes cluster available
- Argo CD installed in namespace `argocd`
- Ingress controller available (Traefik assumed)

### Option A: Argo CD (recommended)

Using the public GitHub manifest URL:

```bash
kubectl apply -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/postgres-stack/argocd/postgres-stack-app.yaml

# Remove Argo CD app + all stack resources
kubectl delete -f https://raw.githubusercontent.com/mtnvencenzo/platform-ops/refs/heads/main/stacks/postgres-stack/argocd/postgres-stack-app.yaml
kubectl delete namespace postgres-platform
```

This deploys to namespace `postgres-platform` and syncs from `stacks/postgres-stack/k8s`.

### Option B: Direct Kustomize apply

From repo root:

```bash
kubectl apply -k stacks/postgres-stack/k8s

# Remove resources applied from this kustomization
kubectl delete -k stacks/postgres-stack/k8s
kubectl delete namespace postgres-platform
```

### Verify

```bash
kubectl -n postgres-platform get pods,svc,ingress,pvc
```

### Access Endpoints

- pgAdmin UI: http://pgadmin.127.0.0.1.sslip.io
- PostgreSQL NodePort: `localhost:30432`

### Default Credentials

- PostgreSQL: `admin` / `password` (database: `postgres`)
- pgAdmin: `admin@admin.com` / `password`

### Quick Connection Examples

```bash
# psql via NodePort
psql -h localhost -p 30432 -U admin -d postgres
```

Connection string:

```text
postgresql://admin:password@localhost:30432/postgres
```

## Stack-Specific Notes

- Known issue: some clients may hit `no pg_hba.conf entry` errors in local setups.
- Local recovery flow from within the Postgres pod:

```bash
echo "host all all 0.0.0.0/0 trust" >> /var/lib/postgresql/data/pg_hba.conf
psql -U admin -d postgres -c "SELECT pg_reload_conf();"
psql -U admin -d postgres -c "SELECT line_number, type, address, auth_method, error FROM pg_hba_file_rules ORDER BY line_number DESC LIMIT 1;"
```

- Use permissive `trust` auth only for local development.

## Configuration Notes

- Credentials are currently defined in `k8s/configmap.yml` (includes a Secret manifest).
- PostgreSQL persistence uses PVC `postgres-data`.
- pgAdmin persistence uses PVC `pgadmin-data`.

## Troubleshooting

```bash
kubectl -n postgres-platform get pods
kubectl -n postgres-platform logs deploy/postgres
kubectl -n postgres-platform logs deploy/pgadmin
```

## Disclaimer

These manifests and instructions are intended for local development, testing, and homelab usage. They are not production-hardened and should be reviewed and adapted before use in shared or production environments.

## References

- https://www.postgresql.org/docs/
- https://www.pgadmin.org/docs/
