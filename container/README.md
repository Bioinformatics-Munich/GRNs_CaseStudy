# Container Image

This directory contains the Singularity definition file for the pySCENIC container used in the analysis workflow.

## pySCENIC Container (`pyscenic_jupyter.sif`)

### Base Image
The container is built on top of the official pySCENIC image provided by the Aerts Lab: `aertslab/pyscenic_scanpy:0.12.1_1.9.1`

### Modifications
Simone Roeh aggregated JupyterLab and Jupyter server components on top of the base pySCENIC image to enable interactive notebook workflows.

### Rebuilding the Container

To rebuild this container, use the provided `Singularity.def` file:

```bash
singularity build pyscenic_jupyter.sif Singularity.def
```

### Workflows Supported

This container is used for the whole analysis workflow:

1. **pySCENIC Preprocessing and Analysis** (`pySCENIC-preprocessing.ipynb` & `pySCENIC-processing.ipynb`)
   - Single-cell RNA-seq preprocessing
   - Dimensionality reduction and clustering
   - SCENIC regulon inference
   - RSS (Regulon Specificity Score) calculation

2. **Manual Cell Type Annotation** (`manual_annotation_sctypedb.ipynb`)
   - Marker-based cell type annotation using ScTypeDB
   - Cluster scoring and visualization
   - Manual annotation assignment

### Usage

To run JupyterLab with the pySCENIC container:

```bash
singularity exec container/pyscenic_jupyter.sif jupyter lab --ip=0.0.0.0 --port=8888 --no-browser
```

### Notes
- Three notebooks (`pySCENIC-preprocessing.ipynb`, `pySCENIC-processing.ipynb`  and `manual_annotation_sctypedb.ipynb`) are designed to run in this container
- The container includes all necessary packages for SCENIC analysis, RSS calculation, and manual annotation workflows
- CSV format is preferred for ScTypeDB files (conversion from Excel can be done within the container if needed)
