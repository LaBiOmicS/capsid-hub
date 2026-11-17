# Maintainer guide

Tasks for people with write access to Capsid Hub.

- [Reviewing pull requests](#reviewing-pull-requests)
- [Repository settings](#repository-settings)
- [Updating the Capsid engine](#updating-the-capsid-engine)
- [Republishing images](#republishing-images)
- [Robot pull requests](#robot-pull-requests)
- [Issue submissions](#issue-submissions)
- [Regenerating the catalog](#regenerating-the-catalog)

---

## Reviewing pull requests

Merge only when the `pr-audit` checks pass and the [review criteria](../CONTRIBUTING.md#review-criteria)
are met. Read `build_commands`, `source` and binary URLs carefully: merging is what allows a recipe to
be built with publishing permissions. Pay particular attention to pull requests that change files
outside `recipes/` and `stacks/`, especially `.github/`.

## Repository settings

- **Branch protection on `main`:** require pull requests, the `pr-audit` checks and at least one
  approving review; disallow force pushes.
- **Actions:** set "Fork pull request workflows" to require approval for first-time contributors, and
  keep the default `GITHUB_TOKEN` permissions read-only (workflows request what they need).
- **Packages:** after the first publication of a new image, set the package visibility to *public* in
  the organisation's package settings and link it to this repository; new GHCR packages are private by
  default.
- **Dependabot:** keep `.github/dependabot.yml` enabled; it proposes updates for the SHA-pinned actions.

## Updating the Capsid engine

All five workflows (`pr-audit`, `issue-submission`, `publish-release`, `friday-robot-sync`, `pages`) install
Capsid from the commit in `env.CAPSID_REF`. To move to a new engine version:

1. Update `CAPSID_REF` to the full commit SHA in **all five** workflows in one pull request.
2. In the same pull request, regenerate the files that depend on the engine:
   ```bash
   uv tool install --force "git+https://github.com/LaBiOmicS/capsid.git@<sha>"
   capsid schema --out-dir schema
   capsid catalog
   capsid docs
   capsid lint .
   ```
   `pr-audit` fails if `schema/` does not match the engine (`capsid schema --check`).
3. After merging, decide whether existing images need republishing (next section). Read the engine's
   changelog: changes to generated Dockerfiles only reach published images when they are rebuilt.

## Republishing images

Run the `publish-release` workflow manually (Actions → *Capsid Hub Container Publisher* → *Run workflow*) with:

- `targets: all` to rebuild every recipe and stack, or
- a space-separated list of paths, such as `recipes/samtools/1.25/recipe.yaml stacks/qc-stack.yaml`.

Republishing pushes the same tags with new digests; users who pinned digests are not affected.
Republish everything after an engine update that fixes image content, after a security update of the
base images or the apt snapshot, or after refreshing lockfiles.

## Robot pull requests

`friday-robot-sync` opens a pull request every Friday with recipes generated from new Bioconda
releases. Before merging:

- Review each recipe as you would a contribution, particularly the test command and license.
- Generate the lockfiles (`capsid lock`) and commit them to the robot's branch.
- Remove recipes that fail or are not wanted before merging.

## Issue submissions

The `issue-submission` workflow builds the recipe from a *New recipe* issue without write permissions
and, if the build and test pass, opens a pull request. Review it as any other pull request, and add the
lockfile before merging.

## Regenerating the catalog

`catalog.json` and `CATALOG.md` are generated. The `publish-release` workflow regenerates and
commits them after every publication, so contributors do not need to. To regenerate them by hand
(for example after an engine update):

```bash
capsid catalog && capsid docs
```

## Website

The website at <https://labiomics.github.io/capsid-hub/> is built by the `pages` workflow with
`capsid site` after every publication (called by `publish-release`) and on demand
(`gh workflow run pages.yaml`). It is not committed. The repository's Pages source must be set to
**GitHub Actions** (Settings → Pages). To preview locally:

```bash
capsid site --hub . --out _site --resolve-digests
python3 -m http.server -d _site 8000
```
