# AccuKnox xBOM Scan Action

Generate a Bill of Materials (BOM) using [knoxctl](https://github.com/accuknox/accuknox-cli-v2), upload results to AccuKnox SaaS, and save the BOM as a downloadable GitHub Actions artifact.

## Supported BOM Types

| Type | Command | Source |
|---|---|---|
| `sbom` | `knoxctl pkgscan` | Filesystem or container image |
| `cbom` | `knoxctl cbom` | Go source code or container image |
| `aibom` | `knoxctl aibom generate` | HuggingFace model |

---

## Setup

Add the following secrets to your repository (**Settings → Secrets and variables → Actions**):

| Secret | Description |
|---|---|
| `ACCUKNOX_TOKEN` | AccuKnox API token → [How to create](https://help.accuknox.com/how-to/how-to-create-tokens/) |
| `ACCUKNOX_ENDPOINT` | AccuKnox SaaS hostname, e.g. `cspm.accuknox.com` |
| `ACCUKNOX_LABEL` | AccuKnox label name → [How to create](https://help.accuknox.com/how-to/how-to-create-labels/) |

---

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `bom-type` | No | `sbom` | `sbom` / `cbom` / `aibom` |
| `path` | No | `.` | Directory to scan (filesystem scans) |
| `image` | No | — | Container image to scan. Used by `cbom` only, e.g. `myapp:latest` |
| `aibom-model` | No | — | HuggingFace model ID, e.g. `google-bert/bert-base-uncased` |
| `token` | **Yes** | — | AccuKnox API token |
| `endpoint` | **Yes** | — | AccuKnox SaaS hostname |
| `label` | **Yes** | — | AccuKnox label name |

---

## Usage Examples

### SBOM — Filesystem

Scans the repository source tree for packages and dependencies.  
> ℹ️ Filesystem scan does **not** scan Docker image contents.

```yaml
- uses: accuknox/xbom-scan-action@v1
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

### CBOM — Go Source Code

Scans Go source code for cryptographic algorithms, protocols, and certificates.

```yaml
- uses: accuknox/xbom-scan-action@v1
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

### CBOM — Container Image

Scans a container image for cryptographic algorithms, protocols, and certificates.

> ⚠️ The build step and scan action **must be in the same job**.

```yaml
- name: Build image
  id: build
  run: |
    IMAGE="myapp:${{ github.sha }}"
    docker build -t "$IMAGE" .
    echo "image=${IMAGE}" >> "$GITHUB_OUTPUT"

- uses: accuknox/xbom-scan-action@v1
  with:
    bom-type:           cbom
    image:              ${{ steps.build.outputs.image }}
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: container
```

---

### AIBOM — HuggingFace Model

Inventories AI/ML model components by fetching metadata from the HuggingFace Hub API.

```yaml
- uses: accuknox/xbom-scan-action@v1
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

      - uses: accuknox/xbom-scan-action@v1
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

After the workflow runs, the generated BOM file is available as a downloadable artifact:

1. Go to your repository on GitHub
2. Click **Actions**
3. Select the workflow run
4. Scroll to the **Artifacts** section
5. Download the BOM file

---

## Publishing BOM to GitHub Releases

If you want to attach the generated BOM file as a GitHub Release asset, use the `anchore/sbom-action/publish-sbom` sub-action alongside this action.

It will detect all BOM artifacts from the workflow run matching the pattern and upload them to the release automatically.

```yaml
- uses: anchore/sbom-action/publish-sbom@v0
  with:
    sbom-artifact-match: ".*\\.json$"
```

> **Important:** The job needs explicit permissions to read artifacts and write to the release:

```yaml
jobs:
  xbom-scan:
    permissions:
      actions:  read
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: accuknox/xbom-scan-action@v1
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

> **Reference:** [anchore/sbom-action — Publishing SBOMs with releases](https://github.com/anchore/sbom-action#publishing-sboms-with-releases)
