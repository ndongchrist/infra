# apps/ — per-service GitOps overlays

The desired state for each service, per environment. Each service's Argo CD
Application (see `../argocd/`, Phase 10) deploys the service's **Helm chart**
(from its own repo) merged with the matching values file here:

```
apps/<service>-service/
├── values-dev.yaml    # 1 Postgres instance, SQLite off, ServiceMonitor off
└── values-prod.yaml   # 3 Postgres instances, replicas+HPA, ServiceMonitor on
```

## CI image-tag bump (GitOps handoff)

Each service repo's `.github/workflows/ci.yml` ends by cloning this repo and
running:

```bash
yq -i '.image.tag = strenv(TAG)' "apps/<service>-service/values-dev.yaml"
```

then committing — so a green build advances `image.tag` to the new git SHA and
Argo CD syncs it. **Never deploys directly.**

### Required secret (per service repo)

`INFRA_TOKEN` — a PAT (or fine-grained token) with `contents:write` on
`ndongchrist/infra`, set as an Actions secret in each `*-service` repo. Without
it the `bump-infra` job can't push here.
