# fork/ci

This is the default branch for the n8n fork. It contains **only** CI tooling and `fork.conf`, and exists to suppress all ~60 upstream GitHub Actions workflows.

## How it works

Setting this orphan branch as the GitHub default branch means:
- **Scheduled workflows** only run from the default branch — `fork/ci` has none
- **Push/PR workflows** reference the default branch's YAML — `fork/ci` has only our custom workflow
- **`workflow_dispatch`** only shows default branch workflows in the Actions UI

The actual n8n code lives on `main-fork`, which is **created automatically by the CI workflow** from the pinned tag + feature branches.

## Configuration: `fork.conf`

All build parameters live in `fork.conf` on this branch:

```bash
PINNED_TAG=n8n@2.10.2                    # upstream release to base on
FEATURE_BRANCHES=(                        # merged in order
  feature/api-share-credentials
  feature/expose-project-fields
)
```

To upgrade n8n or add/remove features, edit `fork.conf` and push to `fork/ci`.

## Building a custom Docker image

### 1. Trigger the GitHub Action

Go to **Actions > "Fork: Build and Push Docker Image"** and click **Run workflow**:

| Input | Description | Default |
|-------|-------------|---------|
| `n8n_version` | Override upstream tag | _(from fork.conf)_ |
| `build_runners` | Also build the task runners image | `true` |
| `push_image` | Push to GHCR (`false` = dry run) | `true` |

The workflow will:
1. Read `fork.conf` from `fork/ci`
2. Checkout the pinned upstream tag
3. Merge each feature branch in order
4. Force-push the result to `main-fork`
5. Build n8n with `build-n8n.mjs`
6. Build and push Docker images to GHCR

### 2. Pull the image

```bash
docker pull ghcr.io/<owner>/n8n:2.10.2-custom
```

## Branch overview

| Branch | Purpose |
|--------|---------|
| `fork/ci` | Default branch. CI config + `fork.conf` (this branch) |
| `main-fork` | Integration branch: auto-created by CI (tag + features merged) |
| `feature/*` | Individual feature branches, rebased on the pinned tag |
| `master` | Tracks upstream `master` (never push to origin) |
