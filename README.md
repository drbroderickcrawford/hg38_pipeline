#!/bin/bash
# ============================================================
# hg38_pipeline: Reproducible setup and PRNP/NEFL extraction
# Author: Dr. Broderick Crawford
# Reference: https://doi.org/10.5281/zenodo.16884639
# ============================================================

# Exit immediately if a command exits with a non-zero status
set -euo pipefail

# ------------------------------
# 1. Setup
# ------------------------------
WORKDIR="$(pwd)"
REF_DIR="${WORKDIR}/reference"
RESULTS_DIR="${WORKDIR}/results"

mkdir -p "$REF_DIR" "$RESULTS_DIR"

echo ">>> Working directory: $WORKDIR"
echo ">>> Reference genome directory: $REF_DIR"
echo ">>> Results directory: $RESULTS_DIR"

# ------------------------------
# 2. Download hg38 reference genome
# ------------------------------
cd "$REF_DIR"
if [ ! -f "hg38.fa.gz" ]; then
    echo ">>> Downloading hg38 reference genome..."
    wget -O hg38.fa.gz "https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz"
else
    echo ">>> hg38.fa.gz already exists, skipping download."
fi

# Unzip reference genome
if [ ! -f "hg38.fa" ]; then
    echo ">>> Unzipping hg38.fa.gz..."
    gunzip -c hg38.fa.gz > hg38.fa
else
    echo ">>> hg38.fa already extracted."
fi

# ------------------------------
# 3. Index the reference genome
# ------------------------------
if [ ! -f "hg38.fa.fai" ]; then
    echo ">>> Indexing reference genome with samtools..."
    samtools faidx hg38.fa
else
    echo ">>> hg38.fa.fai already exists, skipping indexing."
fi

# ------------------------------
# 4. Extract PRNP and NEFL genes
# ------------------------------
cd "$WORKDIR"

# Define coordinates (GRCh38/hg38)
PRNP_REGION="chr20:4699600-4702400"
NEFL_REGION="chr8:24875000-24878000"

echo ">>> Extracting PRNP region: $PRNP_REGION"
samtools faidx "$REF_DIR/hg38.fa" $PRNP_REGION > "$RESULTS_DIR/PRNP.fa"

echo ">>> Extracting NEFL region: $NEFL_REGION"
samtools faidx "$REF_DIR/hg38.fa" $NEFL_REGION > "$RESULTS_DIR/NEFL.fa"

# ------------------------------
# 5. Completion message
# ------------------------------
echo ">>> Extraction complete!"
echo "Results saved in: $RESULTS_DIR"
ls -lh "$RESULTS_DIR"


