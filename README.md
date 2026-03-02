# Mealie Helm Chart

Baseline Helm chart for [Mealie](https://mealie.io) (recipe manager, meal planning, shopping lists). Uses the [bjw-s app-template](https://github.com/bjw-s/app-template).

## Subcharts

| Subchart | Source | Values prefix | Description |
|----------|--------|---------------|-------------|
| **mealie** (app-template) | [bjw-s helm-charts](https://github.com/bjw-s/helm-charts) | `mealie.*` | App template: controllers, persistence, ingress, env (BASE_URL, TZ, etc.). |
| **onepassworditem** | [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm) | `onepassworditem.*` | Optional secrets sync into the release namespace. |

All inputs: **`mealie.*`** (controllers, persistence, ingress, env), **`onepassworditem.enabled`**, **`onepassworditem.items`**. Defaults: see `values.yaml`.

## Configuration reference (all inputs)

Every value is documented below. Source: [bjw-s app-template](https://github.com/bjw-s/helm-charts) (as `mealie.*`) and [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm) (as `onepassworditem.*`). The app-template uses `controllers`, `service`, `ingress`, `persistence`, and `global` / `defaultPodOptions`.

### Subchart: mealie (bjw-s app-template)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `mealie.global.fullnameOverride` | string | — | Override full name (e.g. `mealie`). |
| `mealie.defaultPodOptions.securityContext` | object | — | Default pod security (runAsNonRoot, runAsUser, fsGroup, etc.). |
| `mealie.controllers.main.enabled` | bool | `true` | Enable main controller. |
| `mealie.controllers.main.type` | string | `"deployment"` | Controller type. |
| `mealie.controllers.main.replicas` | int | `1` | Replicas. |
| `mealie.controllers.main.containers.main.image.repository` | string | — | Image (e.g. `ghcr.io/mealie-recipes/mealie`). |
| `mealie.controllers.main.containers.main.image.tag` | string | — | Image tag. |
| `mealie.controllers.main.containers.main.image.pullPolicy` | string | `"IfNotPresent"` | Pull policy. |
| `mealie.controllers.main.containers.main.env` | object | — | Env vars (PUID, PGID, TZ, ALLOW_SIGNUP, BASE_URL, etc.). |
| `mealie.controllers.main.containers.main.probes` | object | — | liveness, readiness, startup (enabled, custom, spec). |
| `mealie.controllers.main.containers.main.resources` | object | — | requests/limits (cpu, memory). |
| `mealie.service.main.enabled` | bool | `true` | Enable main service. |
| `mealie.service.main.ports` | object | — | Ports (e.g. http: 9000). |
| `mealie.ingress.main.enabled` | bool | `true` | Enable ingress. |
| `mealie.ingress.main.hosts` | list | — | Hosts (host, paths, pathType). |
| `mealie.ingress.main.tls` | list | `[]` | TLS (secretName, hosts). |
| `mealie.ingress.main.annotations` | object | `{}` | Ingress annotations. |
| `mealie.persistence.data.enabled` | bool | `true` | Enable data PVC. |
| `mealie.persistence.data.type` | string | `"persistentVolumeClaim"` | Type. |
| `mealie.persistence.data.accessMode` | string | `"ReadWriteOnce"` | Access mode. |
| `mealie.persistence.data.size` | string | — | PVC size (e.g. 10Gi). |
| `mealie.persistence.data.storageClass` | string | — | Storage class. |
| `mealie.persistence.data.existingClaim` | string | — | Use existing PVC. |

### Subchart: onepassworditem

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `onepassworditem.enabled` | bool | `true` | Create OnePasswordItem resources; set `false` if supplying secrets another way. |
| `onepassworditem.defaultVault` | string | `""` | Default vault for items. |
| `onepassworditem.items` | list | `[]` | List of `{ item, name, type }`; `name` must match Secret refs in container env. |

## Chart contents

- **App:** Mealie (ghcr.io/mealie-recipes/mealie) via bjw-s app-template; port 9000.
- **Secrets:** Optional — use onepassworditem for DB or API keys; Mealie stores users in its own SQLite DB.
- **Ingress:** Host/path and TLS; **OIDC** should be enabled at ingress (oauth2-proxy or nginx auth) so the app is not publicly open.
- **Persistence:** Single PVC for `/app/data` (recipes, DB, uploads); chart creates **data** PVC or use `existingClaim`.

## Prerequisites

- **1Password Connect** (optional): if you use `onepassworditem.enabled: true` to sync credentials (e.g. DB, API keys). The chart depends on [expectedbehaviors/OnePasswordItem-helm](https://github.com/expectedbehaviors/OnePasswordItem-helm); secrets are created in the release namespace (`.Release.Namespace`). Set `onepassworditem.items[]` with `{ item, name, type }` and reference the Secret name in your app-template env.

## Requirements

| Dependency | Notes |
|------------|--------|
| **PVC** | Chart creates **data** PVC (default 10Gi); or set `existingClaim`. |
| **Namespace** | e.g. `mealie`; create or use Argo CD project. |
| **OIDC / Auth** | Configure nginx ingress to require auth (oauth2-proxy or nginx auth) so Mealie is not public. |

## Usage

Use this chart with your own values in Argo CD (or `helm install -f my-values.yaml`). Override at least:

- **Domain / ingress:** `mealie.ingress.main.hosts[0].host`, `mealie.ingress.main.tls`, and any ingress annotations (e.g. external-dns, cert-manager).
- **BASE_URL:** `mealie.controllers.main.containers.main.env.BASE_URL` — set to your public URL (e.g. `https://mealie.example.com`) for links and OAuth.
- **TZ:** `mealie.controllers.main.containers.main.env.TZ`.
- **Auth:** Protect the app via OIDC at ingress (e.g. nginx + oauth2-proxy) or disable public signup (`ALLOW_SIGNUP: "false"`) and create an initial user in the app. For automation, create API tokens in Mealie (User profile → API tokens).

## Key values

| Area | Where | What to set |
|------|--------|-------------|
| Base URL | `mealie.controllers.main.containers.main.env.BASE_URL` | Public URL (e.g. `https://mealie.example.com`). |
| Signup | `mealie.controllers.main.containers.main.env.ALLOW_SIGNUP` | `false` (recommended); set `true` only temporarily for first user. |
| Persistence | `mealie.persistence.data` | size, storageClass, or existingClaim. |
| Ingress | `mealie.ingress.main.hosts` | Host and TLS secret for your domain. |

## Values (onepassworditem)

| Key | Description |
|-----|-------------|
| `onepassworditem.enabled` | If `true` (default), the subchart creates OnePasswordItem resources. Set `false` if you supply secrets another way. |
| `onepassworditem.items` | List of `{ item, name, type }`. `name` must match the Secret name used in your container env. |

## Install

**From this repo:**

```bash
helm dependency update
helm install mealie . -f my-values.yaml -n mealie --create-namespace
```

**From Helm repo (expectedbehaviors):**

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/mealie
helm install mealie expectedbehaviors/mealie -f my-values.yaml -n mealie --create-namespace
```

## Render & validation

Run `helm dependency update` first, then:

```bash
helm template mealie . -f my-values.yaml -n mealie
```

## Argo CD

Point your Application at this repo (path: `.`) and pass your values (e.g. from a private values repo or inline). Namespace typically `mealie`.

## Next steps

1. Set **BASE_URL** and create **data** PVC (or existingClaim).
2. Configure **OIDC at ingress** so Mealie is not public.
3. Deploy; create first user; create API tokens for automation if needed.
