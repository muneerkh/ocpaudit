# OCPAudit — OpenShift Cluster Audit Monitor

Kubernetes/OpenShift deployment manifests for **OCPAudit**, a 24/7 cluster
auditing application that answers two questions continuously:

> **Who logged into the cluster — and what changes did they make?**

This repository contains the deployment manifests and container image
references only. The application is distributed as a prebuilt container image.

## Features

- **Login / logout tracking** — detects OAuth token creation/deletion and
  `/oauth/authorize` audit events
- **Per-user API change tracking** — every create/update/patch/delete call with
  username, groups, namespace, resource, object, response code, source IP, and
  raw audit JSON
- **Dual collection paths** — OpenShift Logging `ClusterLogForwarder`
  (full-fidelity apiserver audit logs) + in-cluster Kubernetes watchers
  (OAuth tokens, users, identities, events)
- **PostgreSQL persistence** — batched async writes, configurable retention
  (`OCPAUDIT_RETENTION_DAYS`, `OCPAUDIT_MAX_ROWS`)
- **Web dashboard** — embedded UI with stats cards, filterable/paginated
  records, namespace dropdown, auto-refresh, and expandable raw audit JSON
- **OpenShift OAuth authentication** — UI access restricted to users bound to
  the `cluster-admin` ClusterRole (strict ClusterRoleBinding subject match)
- **Prometheus metrics + Observe integration** — `/metrics` endpoint,
  ServiceMonitor, and a ready-made console dashboard
- **Token redaction** — OAuth token identifiers (`sha256~…`) are never stored
  or displayed

## Container image

| Image | Platform |
|---|---|
| `docker.io/muneerkh/ocpaudit:latest` | `linux/amd64` |

## Requirements

- OpenShift 4.x cluster with `cluster-admin` access
- (Optional) OpenShift Logging 6.x operator for full audit-log forwarding
- User-workload monitoring for Observe dashboards/metrics

## Quick start

```bash
oc new-project ocp-audit

# PostgreSQL (demo deployment, or point DATABASE_URL at your own instance)
oc apply -f deploy/postgres-demo.yaml

# RBAC, service, OAuth client, route, application
oc apply -f deploy/rbac.yaml
oc apply -f deploy/service.yaml
oc apply -f deploy/route.yaml
oc apply -f deploy/oauthclient.yaml
oc apply -f deploy/deployment.yaml

# Full-fidelity audit logs (requires OpenShift Logging 6.x)
oc apply -f deploy/clusterlogforwarder.yaml

# Prometheus scraping + console dashboard
oc apply -f deploy/servicemonitor.yaml
oc apply -f deploy/console-dashboard.yaml
```

The UI is served on the Route host. Sign in with OpenShift OAuth — only users
bound to the `cluster-admin` ClusterRole are granted access.

## Configuration

Key environment variables on the `ocpaudit` Deployment:

| Variable | Default | Description |
|---|---|---|
| `OCPAUDIT_RETENTION_DAYS` | `90` | Prune rows older than N days |
| `OCPAUDIT_MAX_ROWS` | `0` | Hard cap on stored rows (0 = unlimited) |
| `OCPAUDIT_EXCLUDE_CATEGORIES` | unset | Don't store these categories (e.g. `api_change`) |
| `OCPAUDIT_STORE_READS` | `false` | Also persist read-only get/list/watch events |
| `OCPAUDIT_INGEST_TOKEN` | unset | Require bearer token on `/ingest` |
| `OCPAUDIT_ADMIN_ROLE` | `cluster-admin` | ClusterRole required for UI access |

## Record categories

| Category | Meaning |
|---|---|
| `login` | OAuth token minted / authorize request |
| `logout` | OAuth token deleted |
| `api_change` | create/update/patch/delete API call |
| `k8s_event` | Kubernetes Event (watcher) |
| `user_created` / `identity_created` | First-login account provisioning |

## License

Apache License 2.0 — see [LICENSE](LICENSE).
