# OpenObserve Kubernetes Manifests

This directory contains Kubernetes manifests for deploying the OpenObserve stack with OpenTelemetry Collector.

## Contents
- `namespace.yml`: Namespace definition for OpenObserve resources.
- `configmap.yml`: OpenTelemetry Collector configuration.
- `otel-collector.yml`: Deployment and Service for the OpenTelemetry Collector (NodePort for external access).
- `openobserve.yml`: Deployment and Service for OpenObserve (NodePort for external access).
- `ingress.yml`: Ingress for exposing the OpenObserve UI over HTTP.
- `kustomization.yaml`: Kustomize file to manage all resources in this directory.

## Access
- **OpenObserve UI**: http://openobserve.127.0.0.1.sslip.io (via Ingress)
- **OpenTelemetry Collector Endpoints**:
  - OTLP gRPC: NodePort 30417
  - OTLP HTTP: NodePort 30418

## Usage
Apply all resources with Kustomize:
```bash
kubectl apply -k .
```

Or apply individual manifests as needed:
```bash
kubectl apply -f <manifest>.yml
```

---
For more details, see the main stack README.
