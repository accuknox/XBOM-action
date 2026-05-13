<div align="center">

<img src="https://www.accuknox.com/wp-content/uploads/2023/06/accuknox-logo.png" alt="AccuKnox Logo" width="200"/>

# AccuKnox xBOM Scan Action

**Generate, upload, and store Bills of Materials directly from your CI/CD pipeline.**

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-AccuKnox%20xBOM%20Scan-blue?logo=github)](https://github.com/marketplace/actions/accuknox-xbom-scan)
[![knoxctl](https://img.shields.io/badge/powered%20by-knoxctl%20v0.10.0-orange)](https://github.com/accuknox/accuknox-cli-v2)
[![License](https://img.shields.io/badge/license-Apache%202.0-green)](LICENSE)

</div>

---

## Overview

The **AccuKnox xBOM Scan Action** integrates into your GitHub workflow to:

- 🔍 **Scan** source code, container images, Go projects, or AI/ML models
- 📦 **Generate** CycloneDX 1.6 BOMs (SBOM, CBOM, AIBOM)
- ☁️ **Upload** results to AccuKnox SaaS
- 💾 **Save** the BOM as a downloadable GitHub Actions artefact

---

## Supported BOM Types

| Type | Tool | Source | Use Case |
|---|---|---|---|
| `sbom` | `knoxctl pkgscan` | Filesystem or container image | Packages, libraries, dependencies |
| `cbom` | `knoxctl cbom` | Go source or container image | Crypto algorithms, certs, protocols |
| `aibom` | `knoxctl aibom` | HuggingFace model or AWS Bedrock | AI/ML model inventory |

---

## Setup

Add the following under **Settings → Secrets and variables → Actions**.

### Required (all BOM types)

| Secret | Description |
|---|---|
| `ACCUKNOX_TOKEN` | AccuKnox API token. [How to create](https://help.accuknox.com/how-to/how-to-create-tokens/) |
| `ACCUKNOX_ENDPOINT` | AccuKnox endpoint, e.g. `cspm.accuknox.com` |
| `ACCUKNOX_LABEL` | AccuKnox label. [How to create](https://help.accuknox.com/how-to/how-to-create-labels/) |

### Required for AIBOM Bedrock only

| Secret | Description |
|---|---|
| `AWS_ACCESS_KEY_ID` | AWS access key with `bedrock:ListFoundationModels` permission |
| `AWS_SECRET_ACCESS_KEY` | Matching AWS secret access key |

---

## Outputs

| Output | Description |
|---|---|
| `bom-file` | Absolute path to the generated BOM file on the runner |
| `upload-status` | HTTP status code from the AccuKnox SaaS upload |

---

## Usage

### 📦 SBOM from Filesystem

> Scans the repository source tree for packages and dependencies.

```yaml
- uses: accuknox/xbom-action@2.0
  with:
    bom-type:           sbom
    path:               "."
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: application
```

#### Inputs

| Name | Description | Possible Options | Required |
|---|---|---|---|
| `bom-type` | Type of BOM to generate | `sbom` | **Yes** |
| `path` | Directory to scan | Any valid directory path | No (default: `.`) |
| `token` | AccuKnox API token | Token from AccuKnox console | **Yes** |
| `endpoint` | AccuKnox SaaS hostname | Hostname only, no `https://` | **Yes** |
| `label` | AccuKnox label | Label from AccuKnox console | **Yes** |
| `project-name` | AccuKnox project name | Any string | **Yes** |
| `project-classifier` | CycloneDX classifier | `application`, `firmware`, `library` | **Yes** |
| `output-file` | Local BOM file path | File path | No (auto: `<repo>-sbom.json`) |

---

### 🐳 SBOM from Container Image

> Scans a built container image for installed packages. Build the image in the same job; the action only needs the tag.

```yaml
- name: Build image
  id: build
  run: |
    IMAGE="myapp:${{ github.sha }}"
    docker build -t "$IMAGE" .
    echo "image=${IMAGE}" >> "$GITHUB_OUTPUT"

- uses: accuknox/xbom-action@2.0
  with:
    bom-type:           sbom
    image:              ${{ steps.build.outputs.image }}
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: container
```

#### Inputs

| Name | Description | Possible Options | Required |
|---|---|---|---|
| `bom-type` | Type of BOM to generate | `sbom` | **Yes** |
| `image` | Container image reference. Build with any tool (docker, podman, buildah, ko). Build step must run in the same job. | Image tag, e.g. `myapp:abc1234` | **Yes** |
| `token` | AccuKnox API token | Token from AccuKnox console | **Yes** |
| `endpoint` | AccuKnox SaaS hostname | Hostname only, no `https://` | **Yes** |
| `label` | AccuKnox label | Label from AccuKnox console | **Yes** |
| `project-name` | AccuKnox project name | Any string | **Yes** |
| `project-classifier` | CycloneDX classifier | `container` | **Yes** |
| `output-file` | Local BOM file path | File path | No (auto: `<repo>-sbom.json`) |

---

### 🔐 CBOM from Go Source Code

> Scans Go source for cryptographic algorithms, protocols, and certificates.

```yaml
- uses: accuknox/xbom-action@2.0
  with:
    bom-type:           cbom
    path:               "."
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: application
```

#### Inputs

| Name | Description | Possible Options | Required |
|---|---|---|---|
| `bom-type` | Type of BOM to generate | `cbom` | **Yes** |
| `path` | Directory containing Go source | Any valid directory path | No (default: `.`) |
| `token` | AccuKnox API token | Token from AccuKnox console | **Yes** |
| `endpoint` | AccuKnox SaaS hostname | Hostname only, no `https://` | **Yes** |
| `label` | AccuKnox label | Label from AccuKnox console | **Yes** |
| `project-name` | AccuKnox project name | Any string | **Yes** |
| `project-classifier` | CycloneDX classifier | `application`, `library` | **Yes** |
| `output-file` | Local BOM file path | File path | No (auto: `<repo>-cbom.json`) |

---

### 🐳 CBOM from Container Image

> Scans a container image for cryptographic algorithms, protocols, and certificates.
>
> ⚠️ The build step and scan action must be in the same job to share the runner. The action only needs the final image reference. Build with any tool: docker, podman, buildah, ko, etc.

```yaml
- name: Build image
  id: build
  run: |
    IMAGE="myapp:${{ github.sha }}"
    docker build -t "$IMAGE" .
    echo "image=${IMAGE}" >> "$GITHUB_OUTPUT"

- uses: accuknox/xbom-action@2.0
  with:
    bom-type:           cbom
    image:              ${{ steps.build.outputs.image }}
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: container
```

#### Inputs

| Name | Description | Possible Options | Required |
|---|---|---|---|
| `bom-type` | Type of BOM to generate | `cbom` | **Yes** |
| `image` | Container image reference. Build step must run in the same job. | Image tag, e.g. `myapp:abc1234` | **Yes** |
| `token` | AccuKnox API token | Token from AccuKnox console | **Yes** |
| `endpoint` | AccuKnox SaaS hostname | Hostname only, no `https://` | **Yes** |
| `label` | AccuKnox label | Label from AccuKnox console | **Yes** |
| `project-name` | AccuKnox project name | Any string | **Yes** |
| `project-classifier` | CycloneDX classifier | `container` | **Yes** |
| `output-file` | Local BOM file path | File path | No (auto: `<repo>-cbom.json`) |

---

### 🤖 AIBOM from HuggingFace Model

> Inventories an AI/ML model by fetching metadata from the HuggingFace Hub API.

```yaml
- uses: accuknox/xbom-action@2.0
  with:
    bom-type:           aibom
    aibom-source:       huggingface
    aibom-model:        google-bert/bert-base-uncased
    token:              ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:              ${{ secrets.ACCUKNOX_LABEL }}
    project-name:       my-project
    project-classifier: machine-learning-model
```

#### Inputs

| Name | Description | Possible Options | Required |
|---|---|---|---|
| `bom-type` | Type of BOM to generate | `aibom` | **Yes** |
| `aibom-source` | AIBOM data source | `huggingface` | No (default: `huggingface`) |
| `aibom-model` | HuggingFace model ID | e.g. `google-bert/bert-base-uncased`, `meta-llama/Llama-2-7b` | **Yes** |
| `token` | AccuKnox API token | Token from AccuKnox console | **Yes** |
| `endpoint` | AccuKnox SaaS hostname | Hostname only, no `https://` | **Yes** |
| `label` | AccuKnox label | Label from AccuKnox console | **Yes** |
| `project-name` | AccuKnox project name | Any string | **Yes** |
| `project-classifier` | CycloneDX classifier | `machine-learning-model` | **Yes** |
| `output-file` | Local BOM file path | File path | No (auto: `<repo>-aibom.json`) |

---

### 🤖 AIBOM from AWS Bedrock

> Inventories all foundation models accessible in your AWS Bedrock account for the given region. Requires AWS credentials with `bedrock:ListFoundationModels` permission.

```yaml
- uses: accuknox/xbom-action@2.0
  with:
    bom-type:              aibom
    aibom-source:          bedrock
    aws-region:            us-east-1
    aws-access-key-id:     ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    token:                 ${{ secrets.ACCUKNOX_TOKEN }}
    endpoint:              ${{ secrets.ACCUKNOX_ENDPOINT }}
    label:                 ${{ secrets.ACCUKNOX_LABEL }}
    project-name:          my-project
    project-classifier:    machine-learning-model
```

#### Inputs

| Name | Description | Possible Options | Required |
|---|---|---|---|
| `bom-type` | Type of BOM to generate | `aibom` | **Yes** |
| `aibom-source` | AIBOM data source | `bedrock` | **Yes** |
| `aws-region` | AWS region for Bedrock scan | AWS region code, e.g. `us-east-1`, `us-west-2`, `eu-central-1` | **Yes** |
| `aws-access-key-id` | AWS access key ID with `bedrock:ListFoundationModels` permission | AWS access key string | **Yes** |
| `aws-secret-access-key` | AWS secret access key | AWS secret key string | **Yes** |
| `token` | AccuKnox API token | Token from AccuKnox console | **Yes** |
| `endpoint` | AccuKnox SaaS hostname | Hostname only, no `https://` | **Yes** |
| `label` | AccuKnox label | Label from AccuKnox console | **Yes** |
| `project-name` | AccuKnox project name | Any string | **Yes** |
| `project-classifier` | CycloneDX classifier | `machine-learning-model` | **Yes** |
| `output-file` | Local BOM file path | File path | No (auto: `<repo>-aibom.json`) |

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

      - uses: accuknox/xbom-action@2.0
        with:
          bom-type:           sbom
          path:               "."
          token:              ${{ secrets.ACCUKNOX_TOKEN }}
          endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
          label:              ${{ secrets.ACCUKNOX_LABEL }}
          project-name:       my-project
          project-classifier: application
```

---

## Downloading the BOM Artefact

After the workflow runs, the generated BOM is saved as a GitHub Actions artefact:

1. Go to your repository on GitHub
2. Click **Actions**
3. Select the workflow run
4. Scroll to the **Artifacts** section at the bottom
5. Click to download the BOM file

---

## Publishing BOM to GitHub Releases

To attach the BOM as a GitHub Release asset, use the `anchore/sbom-action/publish-sbom` sub-action. It picks up matching artefacts from the workflow run and attaches them to the release.

```yaml
- uses: anchore/sbom-action/publish-sbom@v0
  with:
    sbom-artifact-match: ".*\\.json$"
```

> ⚠️ The job requires explicit permissions:

```yaml
jobs:
  xbom-scan:
    permissions:
      actions:  read
      contents: write

    steps:
      - uses: actions/checkout@v4

      - uses: accuknox/xbom-action@2.0
        with:
          bom-type:           sbom
          path:               "."
          token:              ${{ secrets.ACCUKNOX_TOKEN }}
          endpoint:           ${{ secrets.ACCUKNOX_ENDPOINT }}
          label:              ${{ secrets.ACCUKNOX_LABEL }}
          project-name:       my-project
          project-classifier: application

      - uses: anchore/sbom-action/publish-sbom@v0
        with:
          sbom-artifact-match: ".*\\.json$"
```
