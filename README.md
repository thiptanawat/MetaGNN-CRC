# MetaGNN-CRC Dataset - Data in Brief Repository

**Manuscript:** "MetaGNN-CRC: A Multi-Omics Graph-Tensor Dataset for Patient-Specific
Colorectal Cancer Metabolic Network Reconstruction"

**Journal:** Data in Brief (Elsevier)
**Author:** Thiptanawat Phongwattana
**Corresponding Author:** Jonathan H. Chan (jonathan@sit.kmutt.ac.th)
**Affiliation:** School of Information Technology, King Mongkut's University of
Technology Thonburi (KMUTT), Bangkok 10140, Thailand

---

## Repository Structure

```
DataInBrief_Repository/
├── README.md                                    # This file
├── raw_data/                                    # CSV data underlying each figure
│   ├── dib_fig1_omics_coverage.csv              # Fig 1A: patient omics coverage
│   ├── dib_fig1_stage_distribution.csv          # Fig 1B: AJCC tumour stage
│   ├── dib_fig1_msi_status.csv                  # Fig 1C: MSI classification
│   ├── dib_fig2_rnaseq_vst_distribution.csv     # Fig 2A: VST expression distribution
│   ├── dib_fig2_protein_completeness.csv        # Fig 2B: proteomics completeness curve
│   ├── dib_fig2_recon3d_reaction_coverage.csv   # Fig 2C: reaction coverage by omics
│   └── dataset_manifest.csv                     # Full inventory of all dataset files
├── code/                                        # Full source code
│   ├── requirements.txt                         # Python dependencies
│   ├── 01_preprocess_tcga_rnaseq.py             # RNA-seq VST normalisation pipeline
│   ├── 02_preprocess_cptac_proteomics.py        # TMT proteomics normalisation pipeline
│   ├── 03_construct_hetero_graph.py             # Graph tensor construction
│   ├── 04_generate_dib_figures.py               # All DIB manuscript figures
│   └── 05_validate_dataset.py                   # Dataset integrity checker
├── notebooks/                                   # Jupyter notebooks for exploration
│   ├── 01_Cohort_Overview.ipynb
│   ├── 02_Data_Quality_Assessment.ipynb
│   ├── 02_Data_Quality_Assessment_v2.ipynb
│   └── 03_Dataset_Validation.ipynb
├── results/                                     # Pre-computed results
│   ├── cohort_demographics/
│   ├── dataset_validation/
│   ├── graph_statistics/
│   ├── hma_matching/
│   └── omics_qc/
└── references/
    └── data_access_guide.md                     # Download instructions for all data
```

---

## Quick Start

```bash
# 1. Install dependencies
pip install -r code/requirements.txt

# 2. Reproduce DIB figures from CSV raw data
python code/04_generate_dib_figures.py

# 3. Validate dataset integrity (after downloading full dataset)
python code/05_validate_dataset.py ./path/to/dataset/root

# 4. Run full preprocessing pipeline (requires raw data downloads — see references/)
#    Step A: Preprocess RNA-seq
python code/01_preprocess_tcga_rnaseq.py \
    --gdc_dir ./tcga_star_counts/ \
    --manifest gdc_manifest.txt \
    --recon3d_genes recon3d_gene_list.txt \
    --output_dir ./processed/rnaseq/

#    Step B: Preprocess proteomics
python code/02_preprocess_cptac_proteomics.py \
    --pdc_tsv cptac_crc_tmt_abundance.tsv \
    --pdc_clinical cptac_clinical.tsv \
    --tcga_barcodes tcga_barcodes.txt \
    --recon3d_genes recon3d_gene_list.txt \
    --output_dir ./processed/proteomics/

#    Step C: Build heterogeneous graph tensors
python code/03_construct_hetero_graph.py \
    --recon3d_mat Recon3D.mat \
    --rnaseq_h5 ./processed/rnaseq/tcga_crc_rnaseq_vst.h5 \
    --proteomics_h5 ./processed/proteomics/cptac_crc_protein_tmt.h5 \
    --pubchem_props_tsv pubchem_metabolite_props.tsv \
    --gpr_table_tsv recon3d_gpr_table.tsv \
    --hma_mat Human1_GEMs.mat \
    --output_dir ./graph_data/
```

---

## Dataset Overview

This repository accompanies a Data in Brief manuscript describing a curated,
graph-structured multi-omics dataset for patient-specific colorectal cancer (CRC)
metabolic network reconstruction. The dataset integrates transcriptomic and proteomic
measurements onto the Recon3D v3 human metabolic network as heterogeneous graph tensors,
ready for use with graph neural network (GNN) frameworks.

### Dataset Statistics

| Property | Value |
|----------|-------|
| Total patients | 690 TCGA-CRC (155 COAD + 535 READ cohort) |
| Omics layers | RNA-seq (TCGA STAR counts, VST-normalised) + TMT proteomics (CPTAC) |
| Patients with matched proteomics | 197 |
| Metabolic reference model | Recon3D v3 |
| Reactions | 10,600 |
| Metabolites | 5,835 |
| Genes (GPR-mapped) | 2,248 |
| Stoichiometric edges | 40,425 (20,512 substrate_of + 19,913 produces) |
| Consensus labels | 7,434 active / 3,166 inactive (70.1% active) |
| Pre-training source | 98 Human Metabolic Atlas (HMA) tissue-specific GEMs |
| Licence | CC-BY 4.0 |

---

## Raw Data Description

### Figure 1 — Patient Cohort Overview

`dib_fig1_omics_coverage.csv` — Number of patients per omics modality.

`dib_fig1_stage_distribution.csv` — AJCC tumour stage distribution across the cohort.

`dib_fig1_msi_status.csv` — Microsatellite instability (MSI) classification breakdown.

### Figure 2 — Quality Control Statistics

`dib_fig2_rnaseq_vst_distribution.csv` — Histogram of VST-normalised expression values
across all patients.

`dib_fig2_protein_completeness.csv` — Proteins ranked by fraction of patients with valid
TMT quantification, with completeness threshold applied at 70%.

`dib_fig2_recon3d_reaction_coverage.csv` — Reaction coverage by omics layer
(transcriptomics GPR-mapped vs. proteomics GPR-mapped vs. metabolite node coverage).

### Dataset Manifest

`dataset_manifest.csv` — Complete inventory of all dataset files with filename, format,
approximate size, description, and tensor dimensions.

---

## Reproducibility

All CSV files in `raw_data/` were generated deterministically from the code in `code/`.
To regenerate all figures:

```bash
python code/04_generate_dib_figures.py
```

---

## Licence

Code: MIT Licence
Dataset: Creative Commons Attribution 4.0 International (CC-BY 4.0)

---
