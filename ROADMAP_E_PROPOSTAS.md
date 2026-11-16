# 🧬 Capsid: Registro de Propostas, Arquitetura e Roadmap

Este documento consolida todas as decisões arquiteturais, o status atual de implementação e as propostas futuras para o projeto **Capsid** (o ecossistema sucessor moderno e seguro ao BioContainers/Bioconda).

---

## 1. Visão Geral e Identidade

* **Nome do Projeto:** **Capsid**
* **Slogan:** *The ultra-slim, secure container factory for science.*
* **Inspiração Biológica:** O capsídeo viral é a nano-estrutura biológica perfeita para encapsular, proteger e entregar material genético sem desperdício de espaço.
* **Proposta Central:** Substituir o monólito lento, pesado e centralizado do Bioconda/BioContainers por uma fábrica moderna de containers baseada em **motores Rust (`uv` e `Pixi`)**, imagens multi-stage blindadas por padrão (**não-root e Tini**) e submissão amigável para pesquisadores de todos os níveis.

---

## 2. Estrutura dos Repositórios Locais

Foram criados e versionados **dois repositórios Git independentes** na máquina:

### Repositório 1: `capsid` (Engine e CLI)
* **Caminho:** `/home/dayhoff/workdir/informatics/containers/capsid`
* **Contém:** Código-fonte da CLI em Python/uv, geradores de Dockerfile, sistema de testes, linter de segurança e publicador.
* **Comando instalado no sistema:** `capsid` (disponível em `~/.local/bin/capsid`).

### Repositório 2: `capsid-hub` (Hub de Receitas da Comunidade)
* **Caminho:** `/home/dayhoff/workdir/informatics/containers/capsid-hub`
* **Contém:** Catálogo declarativo de ferramentas (`recipes/<tool>/<version>/recipe.yaml`), esteiras de CI/CD para auditoria e publicação (`.github/workflows/`), template de formulário web para submissões sem Git e índice central `catalog.json`.

---

## 3. O que já está 100% Implementado e Testado

| Funcionalidade | Descrição | Status |
| :--- | :--- | :--- |
| **CLI `capsid`** | Interface de linha de comando moderna, com cores e tabelas Unicode (Rich) | ✔ Implementado |
| **Motor `uv-pypi`** | Compilação ultra-rápida de pacotes Python direto do PyPI | ✔ Testado com `cutadapt` (185 MB) |
| **Motor `pixi-conda`** | Resolução em Rust consumindo Bioconda + Conda-Forge sem o overhead do Conda | ✔ Testado com `bwa` (209 MB) e `fastp` (190 MB) |
| **Exportação Apptainer SIF** | Criação nativa de imagens `.sif` para HPC via flag `--apptainer` | ✔ Testado com `bwa-0.7.19.sif` (47.45 MB) |
| **Multi-Versões** | Suporte a múltiplas versões da mesma ferramenta lado a lado (`samtools@1.19` e `samtools@1.25`) | ✔ Testado |
| **Segurança Padrão (Hardened)** | Usuário `bio:1000` obrigatório, `tini` no PID 1, `/opt` somente leitura, `/tmp` 1777 | ✔ Testado e ativo |
| **Auditor `capsid lint`** | Varredura de segurança, verificação de duplicidade, bloqueio de comandos perigosos e checagem de sha256 | ✔ Implementado |
| **Submissão Fácil (`capsid submit`)** | Criação automática de branch e PR via GitHub CLI ou link de 1 clique web | ✔ Implementado |
| **Formulário Web de Submissão** | Submissão de receitas pelo navegador (Issue Template + Bot conversor para PR) | ✔ Implementado |
| **Sincronização (`capsid sync`)** | Atualização do catálogo e receitas locais a partir do repositório remoto | ✔ Implementado |

---

## 4. Benchmarks Obtidos nos Primeiros Testes

| Ferramenta | Motor | BioContainers Clássico | **Capsid (Hardened)** | **Capsid Apptainer SIF** | Tempo de Build |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`cutadapt:5.2`** | `uv-pypi` | ~1.1 GB | **185.27 MB** *(~82% menor)* | — | 27.8s |
| **`bwa:0.7.19`** | `pixi-conda` | ~850 MB | **209.62 MB** *(~75% menor)* | **47.45 MB** *(~88% menor)* | 18.2s |
| **`fastp:1.4.0`** | `pixi-conda` | ~600 MB | **190.36 MB** *(~68% menor)* | — | 18.2s |

---

## 5. Propostas e Próximos Passos (Implementações Futuras)

Ao retomar os trabalhos, os próximos passos planejados são:

### Proposta A: Stacks Multi-Ferramentas (`capsid compose`)
* **Objetivo:** Permitir que pesquisadores criem containers que combinam 2 ou mais ferramentas (ex: `samtools + bwa + cutadapt` ou `fastqc + multiqc`).
* **Implementação:** Em vez do inferno de conflitos de dependências do Conda, o Capsid usa compilação multi-stage copiando os prefixos limpos de cada ferramenta para uma única imagem runtime segura.

### Proposta B: Publicação e Multi-Arquitetura no GitHub Actions
* **Objetivo:** Ao dar o push dos repositórios no GitHub, ativar a compilação nativa para:
  - `linux/amd64` (Servidores tradicionais e HPC x86)
  - `linux/arm64` (Apple Silicon M1/M2/M3/M4 e nuvem AWS Graviton)
* **Destino:** Publicar automaticamente no GitHub Container Registry (`ghcr.io/capsid/<tool>:<version>`).

### Proposta C: Proveniência e Assinatura Criptográfica (SLSA & Cosign)
* **Objetivo:** Garantir integridade máxima para uso em hospitais, ensaios clínicos e empresas de biotecnologia.
* **Implementação:** Assinar as imagens com **Sigstore / Cosign** e embutir o **SBOM** (*Software Bill of Materials* em formato SPDX ou CycloneDX) dentro da imagem em `/etc/sbom.json`.

### Proposta D: Robô de Atualizações Automáticas (Auto-Updater Bot & Friday Harvester)
* **Status:** ✔ **Implementado!**
* **Objetivo:** Um workflow agendado toda sexta-feira (`cron: '0 4 * * 5'`) que consulta as APIs do Bioconda e BioContainers, identifica novas ferramentas e atualizações, transforma-as em receitas modernas do Capsid com hardening de segurança (`bio:1000`, `tini`, SBOM) e abre automaticamente Pull Requests com relatório completo.
* **Comandos CLI disponíveis:**
  ```bash
  capsid bot check --days 7 --limit 10
  capsid bot sync --days 7 --limit 10 --recipes-dir recipes/
  ```

### Proposta E: Framework Universal de Reprodutibilidade e Replicabilidade
* **Objetivo:** Expandir o Capsid para ser a espinha dorsal de reprodutibilidade científica (ACM/FAIR/ICMJE), incluindo:
  - Determinismo hermético bit-a-bit (`SOURCE_DATE_EPOCH`, lockfiles criptográficos).
  - Proveniência de dados e metadados científicos no padrão **RO-Crate** (`ro-crate-metadata.json`).
  - Suíte de validação científica com *Golden Datasets* e comparação de hashes semânticos de resultados analíticos (BAM, VCF, matrizes de expressão).
  - Assinatura criptográfica (Sigstore / Cosign) e SBOM SPDX/CycloneDX.

### Proposta F: Catálogo Web Interativo e API GA4GH TRS v2
* **Objetivo:** Disponibilizar uma interface web moderna de busca para a comunidade científica navegar pelos containers disponíveis e permitir que orquestradores (como Galaxy, Nextflow Tower e Dockstore) consultem a API nativa do Capsid.

---

## 6. Comandos Rápidos para Quando Você Voltar

### Para testar a CLI localmente:
```bash
capsid --help
capsid catalog
capsid lint recipes/
capsid build bioconda:bedtools@2.31.0
```

### Para conectar e sincronizar com o seu GitHub:
```bash
# Repositório da Engine
cd /home/dayhoff/workdir/informatics/containers/capsid
git remote add origin git@github.com:<sua-org>/capsid.git
git push -u origin main

# Repositório do Hub de Ferramentas
cd /home/dayhoff/workdir/informatics/containers/capsid-hub
git remote add origin git@github.com:<sua-org>/capsid-hub.git
git push -u origin main
```
