# 📚 Capsid Hub Tool Registry

> Ultra-slim, hardened, rootless scientific containers ready for Docker, Apptainer/Singularity, Nextflow, and Snakemake.

**Indexed Tools:** 9 | **Total Recipes/Versions:** 11 | **Specification:** GA4GH TRS v2.0.1 Ready

| Tool | Versions | Engine | Target Architectures | License | Description | Nextflow Snippet | Snakemake Snippet |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`bcftools`** | `1.21`, `1.25` | `pixi-conda` | `amd64, arm64` | `GPL-3.0-or-later` | BCFtools, utilities for manipulating variant calls | `capsid export nextflow bcftools@1.25` | `capsid export snakemake bcftools@1.25` |
| **`bedtools`** | `2.31.1` | `pixi-conda` | `amd64, arm64` | `MIT` | A powerful toolset for genome arithmetic | `capsid export nextflow bedtools@2.31.1` | `capsid export snakemake bedtools@2.31.1` |
| **`bwa`** | `0.7.19` | `pixi-conda` | `amd64, arm64` | `GPL-3.0-or-later` | The BWA read mapper. | `capsid export nextflow bwa@0.7.19` | `capsid export snakemake bwa@0.7.19` |
| **`cutadapt`** | `5.2` | `uv-pypi` | `amd64, arm64` | `MIT` | Adapter trimming and other preprocessing of high-t | `capsid export nextflow cutadapt@5.2` | `capsid export snakemake cutadapt@5.2` |
| **`fastp`** | `1.4.0` | `pixi-conda` | `amd64, arm64` | `MIT` | A ultra-fast FASTQ preprocessor with full features | `capsid export nextflow fastp@1.4.0` | `capsid export snakemake fastp@1.4.0` |
| **`minimap2`** | `2.28` | `pixi-conda` | `amd64, arm64` | `MIT` | A versatile pairwise aligner for genomic and splic | `capsid export nextflow minimap2@2.28` | `capsid export snakemake minimap2@2.28` |
| **`multiqc`** | `1.25.1` | `uv-pypi` | `amd64, arm64` | `GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007` | Create aggregate bioinformatics analysis reports a | `capsid export nextflow multiqc@1.25.1` | `capsid export snakemake multiqc@1.25.1` |
| **`samtools`** | `1.19`, `1.25` | `pixi-conda` | `amd64, arm64` | `MIT` | Tools for dealing with SAM, BAM and CRAM files | `capsid export nextflow samtools@1.25` | `capsid export snakemake samtools@1.25` |
| **`seqkit`** | `2.8.2` | `pixi-conda` | `amd64, arm64` | `MIT` | A cross-platform and ultrafast toolkit for FASTA/Q | `capsid export nextflow seqkit@2.8.2` | `capsid export snakemake seqkit@2.8.2` |

---
## 🚀 Quick Usage in Pipelines

### Nextflow DSL2
```groovy
process SAMTOOLS {
    container 'ghcr.io/labiomics/capsid/samtools:1.25'
    script:
    "samtools view -h in.bam > out.sam"
}
```

### Snakemake
```python
rule bwa_mem:
    container: 'docker://ghcr.io/labiomics/capsid/bwa:0.7.19'
    shell: 'bwa mem {input} > {output}'
```

### Apptainer / HPC Direct
```bash
apptainer exec docker://ghcr.io/labiomics/capsid/samtools:1.25 samtools --version
```
