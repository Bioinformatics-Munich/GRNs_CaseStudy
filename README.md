# SCENIC Analysis Workflow

## Overview
This repository contains a complete SCENIC (Single-Cell Regulatory Network Inference and Clustering) analysis workflow for mouse PBMC 10x Genomics data. The analysis includes preprocessing, SCENIC regulon inference, RSS (Regulon Specificity Score) calculation, and manual cell type annotation.

## Data Structure
- `mouse_1/`: First mouse PBMC sample (10x Genomics format)
- `mouse_2/`: Second mouse PBMC sample (10x Genomics format)
- Key files: `*_feature_bc_matrix.h5` (read directly by scanpy)

### Dataset Download

The mouse PBMC 10x Genomics data was obtained from the 10x Genomics public datasets:

```bash
# Mouse 1 - mouse_1/ folder
curl -O https://cf.10xgenomics.com/samples/cell-exp/6.0.0/SC3_v3_NextGem_DI_CellPlex_Mouse_PBMC_10K_PBMCs_mouse_1/SC3_v3_NextGem_DI_CellPlex_Mouse_PBMC_10K_PBMCs_mouse_1_count_sample_feature_bc_matrix.h5

# Mouse 2 - mouse_2/ folder
curl -O https://cf.10xgenomics.com/samples/cell-exp/6.0.0/SC3_v3_NextGem_DI_CellPlex_Mouse_PBMC_10K_PBMCs_mouse_2/SC3_v3_NextGem_DI_CellPlex_Mouse_PBMC_10K_PBMCs_mouse_2_count_sample_feature_bc_matrix.h5
```

## Getting Started

### 1. Singularity Container

The container is built from the official pySCENIC image with JupyterLab added. See `container/README.md` for details.

**Key packages included:**
- `pyscenic` >= 0.12.0 - Core SCENIC analysis tool
- JupyterLab 4.0.11, Jupyter Server 2.12.1, Jupyter Notebook 7.0.7

### 2. Download SCENIC Databases

**IMPORTANT**: pySCENIC 0.12.0+ requires Feather v2 format databases. Download the following files to the `resources/` directory:

#### Rankings Databases (REQUIRED)

Download the rankings databases from:
```
https://resources.aertslab.org/cistarget/databases/mus_musculus/mm10/refseq_r80/mc_v10_clust/gene_based/
```

TWO rankings files (ending with `*.genes_vs_motifs.rankings.feather`):
- One for **10kb** window (e.g., `mm10-500bp-upstream-10kb-around-tss.genes_vs_motifs.rankings.feather`)
- One for **500bp** window (e.g., `mm10-500bp-upstream-100bp-downstream-tss.genes_vs_motifs.rankings.feather`)

Example download commands:
```bash
cd resources/
wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm10/refseq_r80/mc_v10_clust/gene_based/mm10-500bp-upstream-10kb-around-tss.genes_vs_motifs.rankings.feather
wget https://resources.aertslab.org/cistarget/databases/mus_musculus/mm10/refseq_r80/mc_v10_clust/gene_based/mm10-500bp-upstream-100bp-downstream-tss.genes_vs_motifs.rankings.feather
```

#### Motif Database (REQUIRED)

Download the motif-to-TF mapping file:
```bash
cd resources/
wget https://resources.aertslab.org/cistarget/motif2tf/motifs-v10nr_clust-nr.mgi-m0.001-o0.0.tbl
```

#### Transcription Factor List (REQUIRED)

Download the mouse TF list:
```bash
cd resources/
wget https://resources.aertslab.org/cistarget/tf_lists/allTFs_mm.txt
```

**Note**: The motif database information is embedded in the rankings.feather files. The `.tbl` file provides motif-to-TF mappings used during regulon prediction.

### 3. Workflow Overview

The analysis is performed using two main Jupyter notebooks, both run in the `pyscenic_jupyter.sif` container:

#### Notebook 1: `pySCENIC-preprocessing.ipynb`

This notebook performs the complete preprocessing and SCENIC analysis workflow:

1. **Data Import**: Read 10x Genomics .h5 files for both mouse samples
2. **Loom File Creation**: Convert to loom format for SCENIC analysis
3. **Quality Control**: 
   - Filter cells and genes based on quality metrics
   - Generate QC diagnostic plots
4. **Preprocessing**: 
   - Normalization and log transformation
   - Highly variable gene identification
   - PCA (25 components)
   - UMAP dimensionality reduction
   - Leiden clustering
5. **Marker Gene Identification**: Find top 25 marker genes per cluster using t-test
6. **Batch Effect Assessment**: Visualize and assess batch effects between samples

#### Notebook 2: `pySCENIC-processing.ipynb`

1. **SCENIC Analysis** (run via SLURM batch scripts):
   - **Step 1: GRNBoost2** - Gene regulatory network inference
   - **Step 2: cisTarget** - Regulon prediction
   - **Step 3: AUCell** - Regulon activity scoring
2. **Integration**: Combine scanpy and SCENIC results into a single AnnData object
3. **RSS Calculation**: Calculate Regulon Specificity Scores (RSS) for each regulon-cluster pair
4. **Visualization**: Generate RSS heatmaps (general and rank-based)

#### Notebook 3: `manual_annotation_sctypedb.ipynb`

This notebook performs manual cell type annotation:

1. **Load ScTypeDB**: Read marker gene database from CSV file (`resources/ScTypeDB_full.csv`)
2. **Marker Extraction**: Extract cell type markers (filtered to "Immune system" tissue for PBMC)
3. **Gene Symbol Conversion**: Convert human gene symbols (ScTypeDB) to mouse format
4. **Cluster Scoring**: Score each cluster based on marker gene expression
5. **Auto-Assignment**: Automatically assign cell types based on combined score (expression + marker count)
6. **Manual Review**: Review and manually adjust annotations
7. **Visualization**: Generate UMAP plots and cluster composition heatmaps
8. **Export Results**: Save annotations for downstream analysis

## Resources

- SCENIC Protocol: https://github.com/aertslab/SCENICprotocol
- pySCENIC Documentation: https://pyscenic.readthedocs.io/
- Container Documentation: `container/README.md`

