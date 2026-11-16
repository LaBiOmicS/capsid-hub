# 🌐 Capsid Hub

> **The community registry and recipe hub for Capsid scientific containers.**  
> Ultra-slim, hardened, multi-arch bioinformatics containers ready for HPC, Nextflow, Snakemake, and Kubernetes.

[![Audit Workflow](https://img.shields.io/badge/ci-Capsid%20Bot%20Auditor-success.svg)](.github/workflows/pr-audit.yaml)
[![Registry](https://img.shields.io/badge/registry-ghcr.io%2Fcapsid-blue.svg)](https://github.com/orgs/capsid/packages)
[![Hardened](https://img.shields.io/badge/security-rootless%20%7C%20tini-green.svg)](CONTRIBUTING.md)

---

## 📦 What is Capsid Hub?

Capsid Hub is the open-source repository where scientific container recipes live. Similar to `bioconda-recipes`, but:

1. **Lightweight & Fast**: Individual path-based builds that finish in seconds, avoiding giant monorepo CI queues.
2. **Automated Security & Duplicate Gate**: Every Pull Request is verified automatically by **Capsid Bot** before merge.
3. **Hardened Standards**: Every container runs as unprivileged user `bio:1000` with **Tini** PID 1 init.
4. **Instant OCI Distribution**: Published to `ghcr.io/capsid/<tool>:<version>`.

---

## 🚀 Using Published Images

### With Docker / Podman:

```bash
docker run --rm ghcr.io/capsid/samtools:1.25 samtools --help
docker run --rm ghcr.io/capsid/cutadapt:5.2 cutadapt --version
docker run --rm ghcr.io/capsid/bwa:0.7.19 bwa
```

### With Nextflow:

```groovy
process ALIGN {
    container 'ghcr.io/capsid/bwa:0.7.19'

    input:
    path genome
    path reads

    output:
    path "aligned.bam"

    script:
    """
    bwa mem -t ${task.cpus} ${genome} ${reads} | samtools view -b - > aligned.bam
    """
}
```

### With Apptainer / Singularity:

```bash
apptainer exec docker://ghcr.io/capsid/bwa:0.7.19 bwa
```

---

## 🔍 Catalog Index

| Tool | Version | Engine | Architectures | License | Recipe Path |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`bwa`** | `0.7.19` | `pixi-conda` | `amd64`, `arm64` | GPL-3.0-or-later | [recipes/bwa/0.7.19/recipe.yaml](recipes/bwa/0.7.19/recipe.yaml) |
| **`cutadapt`** | `5.2` | `uv-pypi` | `amd64`, `arm64` | MIT | [recipes/cutadapt/5.2/recipe.yaml](recipes/cutadapt/5.2/recipe.yaml) |
| **`fastp`** | `1.4.0` | `pixi-conda` | `amd64`, `arm64` | MIT | [recipes/fastp/1.4.0/recipe.yaml](recipes/fastp/1.4.0/recipe.yaml) |
| **`samtools`** | `1.19` | `pixi-conda` | `amd64`, `arm64` | MIT | [recipes/samtools/1.19/recipe.yaml](recipes/samtools/1.19/recipe.yaml) |
| **`samtools`** | `1.25` | `pixi-conda` | `amd64`, `arm64` | MIT | [recipes/samtools/1.25/recipe.yaml](recipes/samtools/1.25/recipe.yaml) |

---

## 🤖 Friday Upstream Robot (Bioconda & BioContainers Harvester)

O Capsid Hub conta com um robô automatizado (`.github/workflows/friday-robot-sync.yaml`) que roda **toda sexta-feira às 04:00 UTC**:
1. **Monitoramento Upstream**: Rastreador de novos pacotes e versões lançadas no **Bioconda** e **BioContainers** nos últimos 7 dias.
2. **Transformação Automática**: Converte pacotes legados para o formato Capsid (`uv-pypi` para Python ou `pixi-conda` para binários), gerando imagens até 80% menores.
3. **Hardening & Auditoria**: Aplica padrões de segurança (usuário `bio:1000`, `tini`, SBOM) e valida a receita com o `capsid lint`.
4. **Pull Request Automático**: Abre um PR detalhado com a tabela de ferramentas transformadas e relatório de integridade para a comunidade aprovar.

Você também pode disparar o robô localmente a qualquer momento:
```bash
# Apenas verificar o que há de novo nos últimos 7 dias
capsid bot check --days 7 --limit 10

# Sincronizar e transformar em receitas Capsid
capsid bot sync --days 7 --limit 10 --recipes-dir recipes/
```

---

## 💡 How to Add a Tool

See our [Contribution Guide](CONTRIBUTING.md) to add your favorite tool or version in under 2 minutes.
