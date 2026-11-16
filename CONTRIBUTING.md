# 🤝 Contributing to Capsid Hub

Thank you for contributing to the **Capsid Hub**!  
Capsid is a community-driven, ultra-slim, secure container ecosystem for scientific software.

---

## 📋 The Contribution Lifecycle

```
Fork repo ──> Scaffold recipe ──> Test & Lint locally ──> Open PR ──> Capsid Bot Review ──> Merge & Publish
```

---

## 🛠️ Step-by-Step Guide

### 1. Install Capsid CLI
Ensure you have the latest Capsid CLI installed via [uv](https://docs.astral.sh/uv/):

```bash
uv tool install git+https://github.com/capsid/capsid.git
```

---

### 2. Fork and Clone the Repository

```bash
git clone https://github.com/<your-username>/capsid-hub.git
cd capsid-hub
git checkout -b add-<tool>-<version>
```

---

### 3. Scaffold Your Recipe

Use `capsid init` to automatically discover and scaffold the recipe from upstream:

```bash
# From Bioconda:
capsid init bioconda:<tool>@<version>

# From PyPI (for Python tools):
capsid init pypi:<tool>@<version>

# From GitHub:
capsid init https://github.com/<owner>/<repo>
```

The recipe will be created at:
```
recipes/<tool>/<version>/recipe.yaml
```

---

### 4. Build and Test Locally

Always test the build before opening a PR:

```bash
# Build the container image
capsid build recipes/<tool>/<version>/recipe.yaml

# Run verification test
capsid test recipes/<tool>/<version>/recipe.yaml
```

---

### 5. Run the Security & Quality Linter

Run the Capsid auditor to ensure full compliance with our security policies:

```bash
capsid lint recipes/<tool>/<version>/recipe.yaml
```

The linter validates:
- ✔ **Directory conformance**: `recipes/<tool>/<version>/recipe.yaml`
- ✔ **Supply chain integrity**: `sha256` checksums for source tarballs
- ✔ **Security rules**: Zero `sudo`, no pipes to shell (`| bash`), unprivileged user execution (`bio:1000`)
- ✔ **Test completeness**: Meaningful verification command (not a dummy exit 0)
- ✔ **Immutability check**: Prevents overwriting previously published versions

---

### 6. Submit Your Pull Request

```bash
git add recipes/<tool>/<version>/recipe.yaml
git commit -m "feat(recipe): add <tool> <version>"
git push origin add-<tool>-<version>
```

Open a Pull Request against `main`.  
The **Capsid Bot** will automatically run the audit suite, compile the container, execute tests, and post a verification badge on your PR!
