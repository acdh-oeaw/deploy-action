# acdh-oeaw deploy action

Deploy a Docker image to the ACDH-CH cluster via Rancher API.

## Inputs

## `token`

The Bearer token for the Rancher API. You can create a new token
[here](https://rancher.acdh-dev.oeaw.ac.at/dashboard/account/create-key), and
store it as a GitHub repository secret via Settings > Secrets > Actions > New
repository secret.

## `project`

The Rancher project id.

## `deployment`

The Rancher deployment.

## Example usage

```yaml
# .github/workflows/build-deploy.yml

name: Build and deploy Docker image

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

on:
  push:
    branches:
      - main

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to the Container registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v3
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          # build-args: |
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Deploy to cluster
        uses: acdh-oeaw/deploy-action@v1
        with:
          token: ${{ secrets.RANCHER_API_TOKEN }}
          project: c-zdbh8:p-abcde
          deployment: namespace:web-frontend
```
