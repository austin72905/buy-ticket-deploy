# Observability: Pyroscope + Grafana Alloy

This folder keeps values files for official Grafana Helm charts. The buy-ticket application chart stays focused on the app itself.

## Components

- `pyroscope-values.yaml`: deploys Pyroscope as the profile storage and UI.
- `alloy-values.yaml`: deploys Grafana Alloy and scrapes buy-ticket API Pod `/debug/pprof` endpoints.

## Prerequisites

The buy-ticket API Pods must expose pprof:

```yaml
app:
  pprofEnabled: true
```

`values-local.yaml` already enables this for local K3s testing.

Current Alloy config selects API Pods by labels:

```yaml
app.kubernetes.io/name: buy-ticket
app.kubernetes.io/component: api
```

Scheduler Pods are not scraped in this first version because `APP_ROLE=scheduler` does not start the HTTP server.

## Install

```powershell
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm upgrade --install pyroscope grafana/pyroscope `
  -n observability --create-namespace `
  -f .\observability\pyroscope-values.yaml

helm upgrade --install alloy grafana/alloy `
  -n observability `
  -f .\observability\alloy-values.yaml
```

## Open Pyroscope UI

```powershell
kubectl -n observability port-forward svc/pyroscope 4040:4040
```

Then open:

```text
http://localhost:4040
```

## Validate targets

```powershell
kubectl -n observability get pods
kubectl -n observability logs deploy/alloy
kubectl get pods --show-labels
```

During k6 load tests, query `service_name="buy-ticket-api"` in Pyroscope.