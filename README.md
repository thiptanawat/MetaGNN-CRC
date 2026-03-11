# MetaGNN-CRC Dataset - Data in Brief Repository

**Manuscript:** "MetaGNN-CRC: A Recon3D-Mapped Multi-Omics Dataset for Graph-Based
Metabolic Network Reconstruction in Colorectal Cancer"

**Journal:** Data in Brief (Elsevier)
**Author:** Thiptanawat Phongwattana
**Corresponding Author:** Jonathan H. Chan (jonathan@sit.kmutt.ac.th)
**Affiliation:** School of Information Technology, King Mongkut's University of
Technology Thonburi (KMUTT), Bangkok 10140, Thailand

**Data Archive (Zenodo):** [https://doi.org/10.5281/zenodo.18903519](https://doi.org/10.5281/zenodo.18903519)
**Companion Code (Zenodo):** [https://doi.org/10.5281/zenodo.18903515](https://doi.org/10.5281/zenodo.18903515)

---

## Repository Structure

```
DataInBrief_Repository/
├── README.md                                    # This file
├── MetaGNN_DataInBrief.tex                      # LaTeX manuscript source
├── MetaGNN_DataInBrief.pdf                      # Compiled manuscript
├── MANIFEST.sha256                              # SHA-256 checksums for all data files
├── raw_data/                                    # CSV data underlying figures
├── code/                                        # Full source code
│   ├── requirements.txt                         # Python dependencies
│   ├── 01_preprocess_tcga_rnaseq.py             # RNA-seq TPM processing pipeline
│   ├── 02_preprocess_cptac_proteomics.py        # TMT proteomics processing
│   ├── 03_construct_hetero_graph.py             # Graph tensor construction
│   ├── 04_generate_dib_figures.py               # All DIB manuscript figures
│   └── 05_validate_dataset.py                   # Dataset integrity checker
├── notebooks/                                   # Jupyter notebooks for exploration
├── results/                                     # Pre-computed results
└── references/
    └── data_access_guide.md                     # Download instructions for all data
```

---

## Dataset Overview

This repository contains a curated, graph-structured multi-omics dataset for
patient-specific colorectal cancer (CRC) metabolic network reconstruction. The dataset
integrates transcriptomic and proteomic measurements onto the Recon3D v3 human metabolic
network as heterogeneous graph tensors, directly loadable into PyTorch Geometric.

### Dataset Statistics

| Property | Value |
|----------|-------|
| Total patients | 690 TCGA-CRC (COAD + READ cohorts) |
| Omics layers | RNA-seq (log₂(TPM+1) normalised) + TMT proteomics (CPTAC, z-scored) |
| Patients with matched proteomics | 95 (13.8%) |
| Metabolic reference model | Recon3D v3 |
| Reactions | 10,600 (5,937 with GPR rules; 4,663 without) |
| Metabolites | 5,835 |
| Genes (GPR-mapped) | 2,248 |
| Stoichiometric edges | 40,425 (20,512 substrate_of + 19,913 produces) |
| Consensus labels | 7,434 active / 3,166 inactive (70.1% active) |
| Metabolite features | 519-dimensional (7 physico-chemical + 512 Morgan fingerprint) |
| Licence | CC-BY 4.0 |

### Key Files

| File | Description |
|------|-------------|
| `tcga_rna_seq.csv` | TPM gene expression (40,799 genes × 690 patients) |
| `cptac_proteomics.csv` | z-scored protein abundance (9,153 proteins × 95 patients) |
| `graph_structure.pt` | HeteroData graph (edge indices, 2 relation types) |
| `patient_features/` | 690 per-patient tensor files (scalar GPR-mapped features) |
| `patient_features_v2/` | 690 enriched tensors (3D: mean, max, frac above median) |
| `hma_labels_thresholded.pt` | Expression-thresholded pseudo-labels (recommended) |
| `splits.json` | Train/val/test patient ID assignments |
| `evaluation_subset_ids.json` | 220-patient evaluation subset IDs (for MethodsX companion) |
| `gpr_reaction_mask.pt` | Boolean mask for 5,937 GPR-mapped reactions |
| `proteomics_available_mask.pt` | Boolean mask for 95 patients with proteomics |
| `best_model.pt` | Trained MetaGNN checkpoint (143,489 parameters) |

### Data Quality Notes

- **Proteomic zero-fill:** 595/690 patients lack CPTAC proteomics and receive zero-valued
  proteomic features. This conflates "no measurement" with "zero abundance" and may
  introduce systematic bias. Use `proteomics_available_mask.pt` for sensitivity analyses.
- **Non-GPR reactions:** 4,663/10,600 reactions (44%) lack GPR rules and receive default
  "active" labels. Use `gpr_reaction_mask.pt` to restrict analyses to GPR-mapped reactions.

---

## Quick Start

```bash
# 1. Install dependencies
pip install -r code/requirements.txt

# 2. Validate dataset integrity
python code/05_validate_dataset.py ./path/to/dataset/root

# 3. Load data in Python
import torch
graph = torch.load('data/processed/graph_structure.pt')
patient = torch.load('data/processed/patient_features_v2/PATIENT_ID.pt')
labels = torch.load('data/processed/hma_labels_thresholded.pt')
```

---

## Reproducibility

All data files include SHA-256 checksums in `MANIFEST.sha256` for integrity verification.
A validation script (`code/05_validate_dataset.py`) checks tensor integrity, feature
dimensions, and label distributions.

---

## Licence

Code: MIT Licence
Dataset: Creative Commons Attribution 4.0 International (CC-BY 4.0)

---
