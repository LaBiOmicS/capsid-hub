# Contributing to Capsid Hub

Thank you for packaging a tool. A contribution is a recipe (one tool version) or a stack (several
tools in one image), with its lockfile. After review and merge, the image is built, tested, signed and
published to `ghcr.io/labiomics/capsid` automatically.

To change the engine itself (generated Dockerfiles, CLI), contribute to
[Capsid](https://github.com/LaBiOmicS/capsid/blob/main/CONTRIBUTING.md) instead.

- [Before you start](#before-you-start)
- [Adding a recipe](#adding-a-recipe)
- [Adding a stack](#adding-a-stack)
- [Submitting without Git](#submitting-without-git)
- [Review criteria](#review-criteria)
- [Updating an existing recipe](#updating-an-existing-recipe)

---

## Before you start

- Check [CATALOG.md](CATALOG.md) and open pull requests: the tool or version may already exist.
- Install the Capsid CLI and Docker (with BuildKit) or Podman:
  ```bash
  uv tool install git+https://github.com/LaBiOmicS/capsid.git
  ```
- The recipe format is described in the
  [recipe reference](https://github.com/LaBiOmicS/capsid/blob/main/docs/recipe-reference.md).
  Editors validate recipes as you type with the JSON Schema in `schema/` (VS Code picks it up from
  `.vscode/settings.json`).

## Adding a recipe

```bash
git clone https://github.com/<your-username>/capsid-hub.git
cd capsid-hub
git switch -c add-<tool>-<version>

# 1. Generate the recipe (or write recipes/<tool>/<version>/recipe.yaml by hand)
capsid init bioconda:<tool>@<version>      # conda package
capsid init pypi:<tool>@<version>          # Python package
capsid init github:<owner>/<repo> -v <version>   # source release

# 2. Review it: description, license (SPDX), test command, platforms
$EDITOR recipes/<tool>/<version>/recipe.yaml

# 3. Lock the dependency closure (pixi-conda and uv-pypi recipes)
capsid lock recipes/<tool>/<version>/recipe.yaml

# 4. Lint and build; the build runs the recipe's test
capsid lint recipes/<tool>/<version>/recipe.yaml --catalog catalog.json
capsid build recipes/<tool>/<version>/recipe.yaml

# 5. Commit the recipe and its lockfile
git add recipes/<tool>/<version>/
git commit -m "feat(recipe): add <tool> <version>"
git push origin add-<tool>-<version>
```

Then open a pull request against `main`. `capsid submit recipes/<tool>/<version>/recipe.yaml` does
the fork, branch, commit, push and pull request in one step with the GitHub CLI.

Prefer, in this order: `pixi-conda` when the tool is on Bioconda or conda-forge, `uv-pypi` for pure
Python tools on PyPI, then a source build (`autotools`, `make`, `cmake`, `rust-cargo`), and `binary`
only when no other option exists.

## Adding a stack

Stacks combine recipes that are already in the hub:

```yaml
# stacks/<stack-name>.yaml
name: <stack-name>
version: 1.0.0
description: What the tools are used for together
components:
  - recipe: samtools
    version: "1.25"
  - recipe: bcftools
    version: "1.25"
test:
  command: samtools --version && bcftools --version
```

```bash
capsid lock stacks/<stack-name>.yaml
capsid lint stacks/<stack-name>.yaml
capsid build stacks/<stack-name>.yaml
git add stacks/<stack-name>.*
```

All conda components share one environment and all Python components one virtual environment, so
their versions must be compatible.

## Submitting without Git

Open an issue with the [New recipe](https://github.com/LaBiOmicS/capsid-hub/issues/new?template=new_recipe.yml)
form and paste the recipe. The recipe is linted, built and tested automatically; if that succeeds, a
pull request is opened for a maintainer, who adds the lockfile before merging.

## Review criteria

The pull request checks run `capsid lint` and build and test every changed recipe and stack. Lint
errors and failing tests block the merge. Maintainers also check that:

- **The test is meaningful.** It runs the tool and checks that it works (`tool --version` at minimum;
  better, a small computation with known output). Commands like `true` or `echo` are rejected.
- **Everything is pinned.** Packages pinned with `==`, lockfile committed, `sha256` for every download.
- **Metadata is accurate.** Description, homepage and an SPDX license identifier
  (for example `MIT`, `GPL-3.0-or-later`); the license must allow redistribution.
- **Platforms are real.** List `linux/arm64` only when the tool builds and works there.
- **Build commands are necessary and readable.** `build_commands` are reviewed line by line.

The full list of lint rules is in Capsid's
[security guide](https://github.com/LaBiOmicS/capsid/blob/main/docs/security.md#recipe-linter).

## Updating an existing recipe

- **New upstream version:** add a new directory `recipes/<tool>/<new-version>/`. Published versions
  are kept.
- **Fix to a published version** (wrong test, metadata, missing platform): edit it in place. Merging
  republishes the same tag with a new digest; the linter warns about this (`W040`), and the pull
  request should explain why.
- **Refreshed lockfile** (security fix in a dependency): rerun `capsid lock` and commit the lockfile;
  this also republishes the tag.
