# Security policy

## Reporting a vulnerability

Please **do not** report vulnerabilities in public issues or pull requests. Report them privately
through [Security → Report a vulnerability](https://github.com/LaBiOmicS/capsid-hub/security/advisories/new).

In scope:

- The CI and publishing workflows (for example a pull request or issue that can obtain write
  permissions, secrets, or publish or sign an image).
- A published image that does not match its recipe, or whose signature or attestation is invalid.
- Problems in the Capsid engine itself can be reported here or in
  [LaBiOmicS/capsid](https://github.com/LaBiOmicS/capsid/security/advisories/new).

Vulnerabilities in a packaged tool or its dependencies should be reported to the tool's maintainers.
Once a fixed version is available, open an issue or pull request here so the image is updated (a new
version, or a refreshed lockfile).

We aim to acknowledge reports within 5 working days.

## Verifying images

Every image is signed and attested by this repository's publish workflow. See
[Verifying an image](README.md#verifying-an-image) and Capsid's
[security guide](https://github.com/LaBiOmicS/capsid/blob/main/docs/security.md).
