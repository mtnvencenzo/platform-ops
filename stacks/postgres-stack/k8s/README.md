# Postgres Platform – Kubernetes Manifests (k3d)

Kubernetes manifests for deploying PostgreSQL and pgAdmin to a local k3d cluster.

## Services

| Service | Description | Internal Port | K8s Service |
|---|---|---|---|
| **PostgreSQL** | PostgreSQL 16 (Alpine) | 5432 | `postgres:5432` |
| **pgAdmin** | Web UI for PostgreSQL management | 80 | `pgadmin:5050` |

## Deploy

```bash
kubectl apply -k k8s/
```

Common error:
```
FATAL: no pg_hba.conf entry for host "10.42.0.29", user "admin", database "postgres", no encryption
```
``` shell
# Update pg_hba.config
echo "host all all 0.0.0.0/0 trust" >> /var/lib/postgresql/data/pg_hba.conf

# Reload config
psql -U admin -d postgres -c "SELECT pg_reload_conf();"

# Verify config
psql -U admin -d postgres -c "SELECT line_number, type, address, auth_method, error FROM pg_hba_file_rules ORDER BY line_number DESC LIMIT 1;"
```


## Access

### Web UI (via Ingress)

- **pgAdmin**: http://pgadmin.127.0.0.1.sslip.io:8080


## Cleanup

```bash
kubectl delete namespace postgres-platform
```
