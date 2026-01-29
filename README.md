# NF-Core Pipeline Protocols

![Nextflow](https://img.shields.io/badge/nextflow-%E2%89%A521.10.6-2496ed.svg)
![Singularity](https://img.shields.io/badge/singularity-3.8+-blue.svg)

> Internal documentation and walkthroughs for running nf-core pipelines (ATAC-seq, ChIP-seq) on the lab cluster using Singularity.

## 📖 Table of Contents
- [Prerequisites & Installation](#-prerequisites--installation)
- [Environment Setup](#-environment-setup)
- [Pipeline: ATAC-seq](#-pipeline-atac-seq)
- [Pipeline: ChIP-seq](#-pipeline-chip-seq)
- [Utilities](#-utilities)

## Prerequisites & Installation

We use `micromamba` to manage the Nextflow environment to avoid conflicts.

### 1. Create the Environment
```bash
# Create a fresh environment named 'nf_core_env'
micromamba create -n nf_core_env -c bioconda -c conda-forge nextflow

# Activate the environment
micromamba activate nf_core_env

# Ensure Nextflow is up to date
nextflow self-update
```
### 2. Test the Installation
Before running on real data, ensure the connection to nf-core and Singularity is working.
```bash
nextflow run nf-core/atacseq \
  -profile test,singularity \
  --outdir ./test_results/ \
  -resume
```
### Environment Setup (Crucial): 
Because we are running on NFS, we must explicitly define temporary directories to handle large intermediate files. Run this block before executing any pipeline.

```bash
# Define a writable temp area (ideally node-local scratch if available)
export TMPBASE="/nfs/lab/Luca/NFCore_TMP"

# Create directories if they don't exist
mkdir -p "$TMPBASE" "$TMPBASE/work"
chmod 1777 "$TMPBASE"

# Export Nextflow and Singularity temp variables
export NXF_TEMP="$TMPBASE"
export SINGULARITY_TMPDIR="$TMPBASE"
export TMPDIR="$TMPBASE"
export TMP="$TMPBASE"
export TEMP="$TMPBASE"

# Pass these to the container environment
export SINGULARITYENV_TMPDIR="$TMPBASE"
export SINGULARITYENV_TMP="$TMPBASE"
export SINGULARITYENV_TEMP="$TMPBASE"
```
# Pipeline: ATAC-seq
### 1. Preparation
Check the read length of your fastq files to set the --read_length argument correctly.

```bash
# Check length of first read
zcat /nfs/lab/projects/scleroderma/data/ATAC/BKS_02_S2_L003_R1_001.fastq.gz | awk 'NR%4==2{print length($0); exit}'
```

### 2. Execution
Ensure you have run the Environment Setup step first.


```bash
# Navigate to project directory
cd /nfs/lab/projects/scleroderma/

# Run Pipeline
nextflow run nf-core/atacseq \
  --input /nfs/lab/projects/scleroderma/Assets/SampleINFO_ATACseq.csv \
  --outdir /nfs/lab/projects/scleroderma/analysis/ATAC/ \
  --genome GRCh38 \
  --read_length 100 \
  --narrow_peak \
  -profile singularity \
  -resume
```
Note: The -resume flag is critical. If the connection drops, re-running this command will pick up exactly where it left off.

# Pipeline: ChIP-seq

### 1. Preparation
```bash
# Check length of first read
zcat /nfs/lab/projects/scleroderma/data/data_2025/H3K27Ac_CHIP/example_file.fastq.gz | awk 'NR%4==2{print length($0); exit}'
```
2. Execution
Ensure you have run the Environment Setup step first.

```bash
# Navigate to project directory
cd /nfs/lab/projects/scleroderma/

# Run Pipeline - Broad Peaks
nextflow run nf-core/chipseq \
  --input /nfs/lab/projects/scleroderma/Assets/SampleINFO_CHIPseq.csv \
  --outdir /nfs/lab/projects/scleroderma/analysis/ChIP/ \
  --genome GRCh38 \
  --read_length 100 \
  -profile singularity \
  -work-dir "$TMPBASE/work" \
  -resume

# Run Pipeline - Narrow Peaks
nextflow run nf-core/chipseq \
  --input /nfs/lab/projects/scleroderma/Assets/SampleINFO_CHIPseq.csv \
  --outdir /nfs/lab/projects/scleroderma/analysis/ChIP/ \
  --genome GRCh38 \
  --read_length 100 \
  -profile singularity \
  --narrow_peak \
  -work-dir "$TMPBASE/work" \
  -resume
  ```
