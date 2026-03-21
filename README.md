# workflow-templates

Reusable GitHub/Gitea Actions workflows for `escape-velocity-ventures`.

## Available Workflows

### `reusable-container-build.yml`
Multi-arch Docker image build and push. Supports both GitHub Actions (GHCR, GHA cache) and Gitea Actions (DinD, registry cache).

```yaml
jobs:
  build:
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-container-build.yml@main
    with:
      image-name: my-service
      context: ./src
      dockerfile: ./src/Dockerfile
    secrets: inherit
```

**Gitea DinD example** (insecure registry, Docker CLI install, registry cache):

```yaml
jobs:
  build:
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-container-build.yml@main
    with:
      image-name: my-service
      context: .
      registry: gitea.infrastructure.svc:3000
      registry-org: escape-velocity-ventures
      install-docker-cli: true
      cache-type: registry
      cache-ref: gitea.infrastructure.svc:3000/escape-velocity-ventures/my-service:buildcache
      buildkitd-config: |
        [registry."gitea.infrastructure.svc:3000"]
          http = true
    secrets:
      registry-user: ${{ secrets.REGISTRY_USER }}
      registry-password: ${{ secrets.REGISTRY_TOKEN }}
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `image-name` | string | *required* | Image name (without registry prefix) |
| `context` | string | `.` | Build context path |
| `dockerfile` | string | `Dockerfile` | Path to Dockerfile |
| `platforms` | string | `linux/amd64,linux/arm64` | Target platforms |
| `push` | boolean | `true` | Push to registry (skipped for PRs) |
| `registry` | string | `ghcr.io` | Container registry |
| `registry-org` | string | `escape-velocity-ventures` | Registry organization |
| `extra-tags` | string | `''` | Additional docker/metadata-action tags |
| `buildkitd-config` | string | `''` | BuildKit daemon config (mirrors, insecure registries) |
| `install-docker-cli` | boolean | `false` | Install Docker CLI (for DinD runners) |
| `docker-version` | string | `27.5.1` | Docker CLI version |
| `cache-type` | string | `gha` | Cache backend: `gha` or `registry` |
| `cache-ref` | string | `''` | Registry ref for cache (when `cache-type=registry`) |
| `download-artifact` | string | `''` | Artifact to download before build |
| `artifact-path` | string | `.` | Where to extract the artifact |

**Outputs:** `image-tag` (short SHA), `image-digest`, `image-tags` (all tags)

### `reusable-secrets-scan.yml`
Gitleaks secrets scanning with auto-detection of scan mode (incremental for push/PR, full for schedule/dispatch).

```yaml
jobs:
  scan:
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-secrets-scan.yml@main
```

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `gitleaks-version` | string | `8.21.2` | Gitleaks release version |
| `fail-on-secrets` | boolean | `true` | Fail workflow when secrets detected |

### `reusable-ai-review.yml`
AI-powered PR code review and security review via Claude (Cybill persona).

```yaml
jobs:
  ai-review:
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-ai-review.yml@main
    secrets:
      anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
      app-id: ${{ secrets.TINKERBELLE_APP_ID }}
      app-private-key: ${{ secrets.TINKERBELLE_APP_PRIVATE_KEY }}
```

### `reusable-deploy-staging.yml`
Deploy a container image to Kubernetes staging with bundle validation and Cloudflare cache purge.

```yaml
jobs:
  deploy:
    needs: [build]
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-deploy-staging.yml@main
    with:
      app-name: my-app
      image-name: escape-velocity-ventures/my-app
      image-tag: ${{ needs.build.outputs.image-tag }}
      staging-hostname: staging.my-app.example.com
      zone-id: ${{ vars.ZONE_ID }}
    secrets: inherit
```

### `reusable-stage-production.yml`
Deploy an image to the inactive production slot (blue-green) for preview/testing.

```yaml
jobs:
  stage:
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-stage-production.yml@main
    with:
      app-name: my-app
      image-name: escape-velocity-ventures/my-app
      preview-hostname: preview.my-app.example.com
    secrets: inherit
```

### `reusable-deploy-production.yml`
Blue-green production cutover via Cloudflare tunnel routing with automatic rollback.

```yaml
jobs:
  deploy:
    uses: escape-velocity-ventures/workflow-templates/.github/workflows/reusable-deploy-production.yml@main
    with:
      app-name: my-app
      image-name: escape-velocity-ventures/my-app
      confirm-cutover: ${{ github.event.inputs.confirm_cutover }}
      production-hostname: my-app.example.com
      preview-hostname: preview.my-app.example.com
      tunnel-id: ${{ vars.TUNNEL_ID }}
      zone-id: ${{ vars.ZONE_ID }}
    secrets: inherit
```
