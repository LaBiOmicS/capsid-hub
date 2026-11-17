# Capsid Hub

**Community recipes and published images for [Capsid](https://github.com/LaBiOmicS/capsid):
reproducible, hardened, signed multi-architecture containers for bioinformatics tools.**

[![PR audit](https://github.com/LaBiOmicS/capsid-hub/actions/workflows/pr-audit.yaml/badge.svg)](https://github.com/LaBiOmicS/capsid-hub/actions/workflows/pr-audit.yaml)
[![Publish](https://github.com/LaBiOmicS/capsid-hub/actions/workflows/publish-release.yaml/badge.svg)](https://github.com/LaBiOmicS/capsid-hub/actions/workflows/publish-release.yaml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

This repository holds the recipes (`recipes/<tool>/<version>/`), multi-tool stacks (`stacks/`), their
lockfiles, and the pipeline that builds, tests, signs and publishes every image to
`ghcr.io/labiomics/capsid`.

- **Browse the tools:** the [hub website](https://labiomics.github.io/capsid-hub/) has a searchable
  catalogue and one page per image (digest, locked closure, usage, verification); also
  [CATALOG.md](CATALOG.md) and the machine-readable [catalog.json](catalog.json).
- **Guides:** [using the images](https://labiomics.github.io/capsid-hub/guide/using/index.html) and
  [submitting a tool](https://labiomics.github.io/capsid-hub/guide/contributing/index.html).
- **Add a tool or version:** [CONTRIBUTING.md](CONTRIBUTING.md).
- **Report a security issue:** [SECURITY.md](SECURITY.md).

---

## Contents

- [Using the images](#using-the-images)
- [What every image provides](#what-every-image-provides)
- [Verifying an image](#verifying-an-image)
- [How publishing works](#how-publishing-works)
- [Upstream robot](#upstream-robot)
- [Repository layout](#repository-layout)

---

## Using the images

```bash
docker run --rm ghcr.io/labiomics/capsid/samtools:1.25 samtools --version
apptainer exec docker://ghcr.io/labiomics/capsid/samtools:1.25 samtools --version
```

Images are tagged with the tool version, and stacks with the stack version:

```
ghcr.io/labiomics/capsid/<tool>:<version>
ghcr.io/labiomics/capsid/<stack>:<version>
```

Nextflow:

```groovy
process {
    withName: 'SAMTOOLS_.*' {
        container = 'ghcr.io/labiomics/capsid/samtools:1.25'
    }
}
```

Snakemake:

```python
rule sort:
    container: "docker://ghcr.io/labiomics/capsid/samtools:1.25"
    shell: "samtools sort -o {output} {input}"
```

Ready-made modules, rules and configurations can be generated with the Capsid CLI
(`capsid export nextflow samtools@1.25`); see Capsid's
[workflow integration guide](https://github.com/LaBiOmicS/capsid/blob/main/docs/workflow-integration.md).
For reproducible pipelines, reference images by digest (`…/samtools@sha256:…`).

## What every image provides

- `linux/amd64` and, when the recipe supports it, `linux/arm64`, under one multi-architecture tag.
- A numeric non-root user (`1000:1000`), `tini` as PID 1 and read-only tool directories.
- The complete dependency closure pinned by lockfile; base images and Debian packages pinned by digest
  and snapshot date.
- Embedded metadata in `/etc/capsid/`: the recipe, the lockfile, an SPDX SBOM, RO-Crate metadata, a
  Nextflow module and a Snakemake rule.
- A Cosign signature, an SBOM attestation and SLSA build provenance.
- A test run on native amd64 and arm64 machines after publication.

Details: Capsid's [image design](https://github.com/LaBiOmicS/capsid/blob/main/docs/image-design.md)
and [reproducibility](https://github.com/LaBiOmicS/capsid/blob/main/docs/reproducibility.md) guides.

## Verifying an image

```bash
IMAGE=ghcr.io/labiomics/capsid/samtools:1.25

cosign verify "$IMAGE" \
  --certificate-identity-regexp '^https://github.com/LaBiOmicS/capsid-hub/\.github/workflows/publish-release\.yaml@' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com

gh attestation verify "oci://$IMAGE" --repo LaBiOmicS/capsid-hub
```

SBOM attestations, vulnerability scanning and admission control are covered in Capsid's
[security guide](https://github.com/LaBiOmicS/capsid/blob/main/docs/security.md#verifying-a-published-image).

## How publishing works

| Step | Workflow | Permissions |
| :--- | :--- | :--- |
| A pull request adds or changes a recipe or stack | `pr-audit`: schema check, `capsid lint`, build and test of each changed file | read-only |
| A recipe is submitted as an issue (form *New recipe*) | `issue-submission`: the recipe is built and tested without write access; if it passes, a pull request is opened | the build job is read-only |
| A maintainer reviews and merges | | |
| The merge reaches `main` | `publish-release`: build for every platform, test, push, sign, attest, then pull and test on native amd64 and arm64 runners | `packages: write`, `id-token: write` |

The workflows install the Capsid engine from a fixed commit (`CAPSID_REF`). Maintainer tasks are
described in [docs/maintainers.md](docs/maintainers.md).

## Upstream robot

Every Friday at 04:00 UTC, `friday-robot-sync` checks Bioconda for tools and versions released in the
previous week, generates Capsid recipes for them, lints them, and opens a pull request with a report.
Nothing is published until a maintainer reviews and merges that pull request. Existing recipes are
never modified by the robot.

The same check can be run locally:

```bash
capsid bot check --days 7 --limit 10                       # list candidates
capsid bot sync --days 7 --limit 10 --recipes-dir recipes  # write recipes
```

## Repository layout

```
recipes/<tool>/<version>/recipe.yaml       recipe
recipes/<tool>/<version>/pixi.lock         lockfile (pixi-conda)
recipes/<tool>/<version>/requirements.lock lockfile (uv-pypi)
stacks/<stack>.yaml                        stack, with <stack>.pixi.lock / <stack>.requirements.lock
schema/                                    JSON Schema of the recipe and stack formats
catalog.json, CATALOG.md                   generated catalog (the website is built by CI, not committed)
.github/workflows/                         CI and publishing
```

## License

Recipes and workflows in this repository are licensed under the [MIT License](LICENSE). Each packaged
tool keeps its own license, recorded in its recipe and in the image labels.
