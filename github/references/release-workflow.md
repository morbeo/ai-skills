# Release Automation Workflow Reference

Full workflow triggered on `v*` tag push.

---

## Python (PyPI) + GitHub Release

```yaml
name: Release

on:
  push:
    tags: ["v*"]

permissions:
  contents: write   # Create GitHub Release
  id-token: write   # Trusted publishing to PyPI

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.tag }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0      # Full history for changelog

      - name: Get version
        id: version
        run: echo "tag=${GITHUB_REF_NAME}" >> $GITHUB_OUTPUT

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Build
        run: |
          pip install hatchling build
          python -m build

      - name: Upload dist
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  release:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Download dist
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Generate changelog
        id: changelog
        run: |
          pip install git-cliff
          git cliff --tag ${{ needs.build.outputs.version }} --unreleased --strip all > notes.md

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          body_path: notes.md
          files: dist/*

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        # Trusted publishing — no PYPI_TOKEN needed
        # Requires PyPI project configured with GitHub OIDC trusted publisher
```

---

## Docker + GHCR

```yaml
name: Docker Release

on:
  push:
    tags: ["v*"]

permissions:
  contents: read
  packages: write

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/metadata-action@v5
        id: meta
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}

      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## npm

```yaml
- name: Publish to npm
  run: npm publish --provenance --access public
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Add `--provenance` for npm's supply chain attestation (requires `id-token: write`).

---

## PyPI Trusted Publishing Setup

In PyPI project settings → Publishing → Add a publisher:
- Owner: `your-org`
- Repo: `your-repo`
- Workflow: `release.yml`
- Environment: (leave blank, or match workflow environment name)

This replaces `PYPI_TOKEN` entirely with OIDC. No long-lived secret needed.

---

## Tagging Workflow (local)

```bash
# Bump version in pyproject.toml / package.json first, then:
git add -A
git commit -m "chore: release v1.2.3"
git tag v1.2.3
git push origin main --tags
```

Or with gh CLI:
```bash
gh release create v1.2.3 --generate-notes --draft
# Review draft in UI, then publish
```
