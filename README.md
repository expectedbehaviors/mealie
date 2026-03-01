# Mealie Helm Chart

Baseline Helm chart for [Mealie](https://mealie.io) (recipe manager, meal planning, shopping lists). Uses the [bjw-s app-template](https://github.com/bjw-s/app-template).

## Usage

Use this chart with your own values in ArgoCD (or `helm install -f my-values.yaml`). Override at least:

- **Domain / ingress:** `mealie.ingress.main.hosts[0].host`, `mealie.ingress.main.tls`, and any ingress annotations (e.g. external-dns, cert-manager).
- **BASE_URL:** `mealie.controllers.main.containers.main.env.BASE_URL` — set to your public URL for links and OAuth.
- **TZ:** `mealie.controllers.main.containers.main.env.TZ`.
- **Auth:** Protect the app via OIDC at ingress (e.g. nginx + oauth2-proxy) or disable public signup and create an initial user in the app.

## Install

```bash
helm dependency update
helm install mealie . -f my-values.yaml -n mealie --create-namespace
```

## ArgoCD

Point your Application at this repo and pass your values (e.g. from a private values repo or inline).
