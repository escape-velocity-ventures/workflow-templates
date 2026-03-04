# workflow-templates

Reusable GitHub Actions workflows for `escape-velocity-ventures`.

## Available Workflows

### `reusable-container-build.yml`
Multi-arch Docker image build and push to GHCR (linux/amd64 + linux/arm64).

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
