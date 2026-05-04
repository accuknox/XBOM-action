<div align="center">

<img src="https://www.accuknox.com/wp-content/uploads/2023/06/accuknox-logo.png" alt="AccuKnox Logo" width="200"/>

# AccuKnox xBOM Scan Action

**Generate, upload, and store Bills of Materials directly from your CI/CD pipeline**

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-AccuKnox%20xBOM%20Scan-blue?logo=github)](https://github.com/marketplace/actions/accuknox-xbom-scan)
[![knoxctl](https://img.shields.io/badge/powered%20by-knoxctl%20v0.10.0-orange)](https://github.com/accuknox/accuknox-cli-v2)
[![License](https://img.shields.io/badge/license-Apache%202.0-green)](LICENSE)

</div>

---

## Overview

The **AccuKnox xBOM Scan Action** integrates seamlessly into your GitHub workflow to:

- 🔍 **Scan** your source code, Go projects, or AI/ML models
- 📦 **Generate** CycloneDX-compliant BOMs (SBOM, CBOM, AIBOM)
- ☁️ **Upload** results directly to AccuKnox SaaS
- 💾 **Save** the BOM as a downloadable GitHub Actions artifact

---

## Supported BOM Types

| Type | Tool | Source | Use Case |
|---|---|---|---|
| `sbom` | `knoxctl pkgscan` | Filesystem | Packages, libraries, dependencies |
| `cbom` | `knoxctl cbom` | Go source or container image | Crypto algorithms, certs, protocols |
| `aibom` | `knoxctl aibom` | HuggingFace model | AI/ML model inventory |

---

## Setup

Add the following to your repository under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `ACCUKNOX_TOKEN` | AccuKnox API token → [How to create](https://help.accuknox.com/how-to/how-to-create-tokens/) |
| `ACCUKNOX_ENDPOINT` | AccuKnox SaaS hostname, e.g. `cspm.accuknox.com` |
| `ACCUKNOX_LABEL` | AccuKnox label → [How to create](https://help.accuknox.com/how-to/how-to-create-labels/) |

---

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `bom-type` | No | `sbom` | `sbom` / `cbom` / `aibom` |
| `path` | No | `.` | Directory to scan |
| `image` | No | — | Container image to scan (`cbom` only) |
| `aibom-model` | No | — | HuggingFace model ID (`aibom` only) |
| `cbom-plugins` | No | — | Comma-separated plugins for `cbom image` scan, e.g. `certificates,keys` |
| `token` | **Yes** | — | AccuKnox API token |
| `endpoint` | **Yes** | — | AccuKnox SaaS hostname |
| `label` | **Yes** | — | AccuKnox label name |

---

## Usage

### 📦 SBOM — Filesystem

> Scans the repository source tree for packages and dependencies.

```yaml
- uses: accuknox/xbom-scan-action@v1.0
  with:
    bom-type:           sbom
    path:               "."
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: container
```

---

### 🔐 CBOM — Go Source Code

> Scans Go source code for cryptographic algorithms, protocols, and certificates.

```yaml
- uses: accuknox/xbom-scan-action@v1.0
  with:
    bom-type:           cbom
    path:               "."
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: application
```

---

### 🐳 CBOM — Container Image

> Scans a container image for cryptographic algorithms, protocols, and certificates.
>
> ⚠️ The build step and scan action **must be in the same job** to share the same runner. The action only needs the final image reference — build it any way you want (`docker`, `podman`, `buildah`, `ko`, etc.).

```yaml
# Build your image however you want — just expose the tag as a step output
- name: Build image
  id: build
  run: |
    IMAGE="myapp:${{ github.sha }}"
    docker build -t "$IMAGE" .          # replace with your build tool
    echo "image=${IMAGE}" >> "$GITHUB_OUTPUT"

- uses: accuknox/xbom-scan-action@v1.0
  with:
    bom-type:           cbom
    image:              ${{ steps.build.outputs.image }}
    cbom-plugins:       certificates,keys    # optional — omit to scan all
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: container
```

---

### 🤖 AIBOM — HuggingFace Model

> Inventories AI/ML model components by fetching metadata from the HuggingFace Hub API.

```yaml
- uses: accuknox/xbom-scan-action@v1.0
  with:
    bom-type:           aibom
    aibom-model:        google-bert/bert-base-uncased
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: machine-learning-model
```

---

## Complete Workflow Example

```yaml
name: AccuKnox xBOM Scan

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  xbom-scan:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: accuknox/xbom-scan-action@v1.0
        with:
          bom-type:           sbom
          path:               "."
          token:              ${{ secrets.ACCUKNOX_TOKEN }}
          endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
          label:              ${{ secrets.ACCUKNOX_LABEL }}
          project-name:       my-project
          project-classifier: container
```

---

## Downloading the BOM Artifact

After the workflow runs the generated BOM is saved as a GitHub Actions artifact:

1. Go to your repository on GitHub
2. Click **Actions**
3. Select the workflow run
4. Scroll to the **Artifacts** section at the bottom
5. Click to download the BOM file

---

## Publishing BOM to GitHub Releases

To attach the BOM as a GitHub Release asset, use the `anchore/sbom-action/publish-sbom` sub-action. It automatically picks up all matching artifacts from the workflow run and attaches them to the release.

```yaml
- uses: anchore/sbom-action/publish-sbom@v0
  with:
    sbom-artifact-match: ".*\\.json$"
```

> ⚠️ **The job requires explicit permissions:**

```yaml
jobs:
  xbom-scan:
    permissions:
      actions:  read
      contents: write

    steps:
      - uses: actions/checkout@v4

      - uses: accuknox/xbom-scan-action@v1.0
        with:
          bom-type:           sbom
          path:               "."
          token:              ${{ secrets.ACCUKNOX_TOKEN }}
          endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
          label:              ${{ secrets.ACCUKNOX_LABEL }}
          project-name:       my-project
          project-classifier: container

      - uses: anchore/sbom-action/publish-sbom@v0
        with:
          sbom-artifact-match: ".*\\.json$"
```

> 📖 Reference: [anchore/sbom-action — Publishing SBOMs with releases](https://github.com/anchore/sbom-action#publishing-sboms-with-releases)

---

<div align="center">

Made with ❤️ by [AccuKnox](https://www.accuknox.com)

</div>
