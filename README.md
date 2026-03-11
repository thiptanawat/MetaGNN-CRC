# MetaGNN-CRC Dataset

**Manuscript:** "MetaGNN-CRC: A Recon3D-Mapped Transcriptomic Dataset with Proteomics-Ready Architecture for Graph-Based Metabolic Network Reconstruction in Colorectal Cancer"

**Journal:** Data in Brief (Elsevier)
**Authors:** Thiptanawat Phongwattana, Jonathan H. Chan*
**Affiliation:** School of Information Technology, King Mongkut's University of Technology Thonburi (KMUTT), 126 Pracha Uthit Rd., Bang Mod, Thung Khru, Bangkok 10140, Thailand
*Corresponding author: jonathan@sit.kmutt.ac.th

**Data Archive (Zenodo):** [https://doi.org/10.5281/zenodo.18903519](https://doi.org/10.5281/zenodo.18903519)
**Companion Code (Zenodo):** [https://doi.org/10.5281/zenodo.18903515](https://doi.org/10.5281/zenodo.18903515)
**Companion Methods Paper:** [MetaGNN (MethodsX)](https://github.com/thiptanawat/MetaGNN)

---

## Repository Structure

```
MetaGNN-CRC/
├── README.md                                    # This file
├── MetaGNN_DataInBrief.tex                      # LaTeX manuscript source
├── MetaGNN_DataInBrief.pdf                      # Compiled manuscript
├── fig1_data_pipeline.png                       # Figure 1: Data processing pipeline
├── MANIFEST.sha256                              # SHA-256 checksums for all data files
├── data/processed/
│   ├── tcga_rna_seq.csv                         # TPM gene expression (40,799 genes × 690 patients)
│   ├── cptac_proteomics.csv                     # Placeholder (zero-filled; see Limitations)
│   ├── graph_structure.pt                       # HeteroData graph (10,600 rxn + 5,835 met nodes)
│   ├── shared_metabolite.pt                     # Full shared_metabolite edge index (7.5M edges)
│   ├── patient_features/                        # 690 per-patient tensors (v1: scalar GPR-mapped)
│   ├── patient_features_v2/                     # 690 per-patient tensors (v2: 3D enriched)
│   ├── hma_labels.pt                            # HMA bounds (all 10,600 active; NOT recommended)
│   ├── hma_labels_thresholded.pt                # Expression-thresholded labels (7,434/3,166; RECOMMENDED)
│   ├── best_model.pt                            # Trained MetaGNN checkpoint (143,489 params, 576 KB)
│   ├── splits.json                              # Train/val/test patient ID assignments (690 patients)
│   ├── evaluation_subset_ids.json               # 220-patient evaluation subset IDs
│   ├── metadata.csv                             # Patient metadata (ID, cohort, split)
│   ├── results_summary.json                     # Training and test metrics
│   ├── stage2_log.csv                           # Per-epoch training log (200 epochs)
│   ├── depmap_evaluation.json                   # DepMap CRISPR evaluation (61 CRC cell lines)
│   ├── subsystem_mapping.json                   # Reaction-to-subsystem mapping (88 subsystems)
│   ├── gpr_reaction_mask.pt                     # Boolean mask: 5,937 GPR-mapped reactions
│   └── proteomics_available_mask.pt             # Boolean mask (all zeros in this release)
├── code/
│   ├── requirements.txt                         # Python dependencies
│   ├── 01_preprocess_tcga_rnaseq.py             # RNA-seq TPM processing pipeline
│   ├── 02_preprocess_cptac_proteomics.py        # Proteomics processing (architecture-ready)
│   ├── 03_construct_hetero_graph.py             # Graph tensor construction
│   ├── 04_generate_dib_figures.py               # All DIB manuscript figures
│   └── 05_validate_dataset.py                   # Dataset integrity checker
├── notebooks/                                   # Jupyter notebooks for exploration
└── references/
    └── data_access_guide.md                     # Download instructions for all data sources
```

---

## Dataset Overview

MetaGNN-CRC is a curated, Recon3D-mapped transcriptomic dataset with proteomics-ready architecture for graph-based metabolic network reconstruction in colorectal cancer. It provides RNA-seq data for 690 TCGA colorectal adenocarcinoma patients pre-processed and mapped to the Recon3D v3 metabolic network as heterogeneous graph-structured tensors, directly loadable into PyTorch Geometric.

**Value:** No existing public dataset bridges the gap between genome-scale metabolic models (GEMs) and graph neural networks. Preparing this data from scratch requires 2-4 weeks of specialised bioinformatics work. MetaGNN-CRC eliminates this bottleneck.

### Dataset Statistics

| Property | Value |
|----------|-------|
| Total patients | 690 TCGA-CRC (COAD + READ cohorts) |
| Transcriptomics | RNA-seq TPM (40,799 genes × 690 patients, 178 MB) |
| Proteomics | Zero-filled for all 690 patients (architecture-ready; see Limitations) |
| Reference model | Recon3D v3 |
| Reaction nodes | 10,600 (5,937 with GPR rules; 4,663 without) |
| Metabolite nodes | 5,835 |
| Genes (GPR-mapped) | 2,248 |
| Stoichiometric edges | 40,425 (20,512 substrate_of + 19,913 produces) |
| Shared_metabolite edges | 7,517,742 undirected (~83,306 at default k=10 sparsification) |
| Metabolite features | 519-dimensional (7 physico-chemical + 512-bit Morgan fingerprints) |
| Expression-thresholded labels | 7,434 active / 3,166 inactive (70.1% / 29.9%) |
| Model checkpoint | 143,489 parameters, 576 KB |
| Licence | CC-BY 4.0 |

### Label Files

Three label sets are associated with this dataset. **Use `hma_labels_thresholded.pt` for training.**

| File / Source | Active / Inactive | Notes |
|---------------|-------------------|-------|
| `hma_labels.pt` | 10,600 / 0 | Generic HMA bounds (all active); NOT recommended for training |
| `hma_labels_thresholded.pt` | 7,434 / 3,166 | Expression-thresholded consensus (RECOMMENDED); "hma" prefix is historical artefact |
| 11-tissue union (code-generated) | 8,147 / 2,453 | Generated dynamically by companion code; not deposited as a file |

### Feature Versions

| Version | Dimensions | Description |
|---------|-----------|-------------|
| v1 (`patient_features/`) | 10,600 × 2 | Scalar GPR-mapped expression + zero-filled proteomic channel |
| v2 (`patient_features_v2/`) | 10,600 × 3 | Enriched: mean expression, max expression, fraction above cohort median |

The trained model checkpoint uses v2 features.

---

## Quick Start

```python
import torch, json

# Load graph structure (HeteroData uses pickle; only load from trusted sources)
graph = torch.load('data/processed/graph_structure.pt')

# Load patient features (weights_only=True for simple tensors, PyTorch >= 2.0)
patient = torch.load('data/processed/patient_features_v2/TCGA-A6-2671.pt',
                     weights_only=True)

# Load labels
labels = torch.load('data/processed/hma_labels_thresholded.pt',
                    weights_only=True)

# Load model checkpoint
model_ckpt = torch.load('data/processed/best_model.pt',
                        map_location='cpu', weights_only=True)

# Verify shapes
assert graph['reaction'].num_nodes == 10600
assert graph['metabolite'].num_nodes == 5835
assert patient.shape[0] == 10600   # reactions
assert patient.shape[1] == 3       # enriched features (v2)
assert labels.shape[0] == 10600
assert labels.sum().item() == 7434 # active reactions

print("All checks passed. Dataset is ready for use.")
```

### Validate Dataset Integrity

```bash
pip install -r code/requirements.txt
python code/05_validate_dataset.py ./data/processed/
```

---

## Limitations

1. **Pseudo-labels, not ground truth.** Expression-thresholded labels are surrogate labels derived from gene expression via GPR rules and cohort majority vote, not experimentally measured reaction fluxes. Users with higher-quality supervision (e.g., ¹³C flux measurements, DepMap essentiality) should replace them.

2. **Proteomic channel zero-filled.** Two CPTAC studies exist: PDC000111 (TCGA Retrospective, ~90 patients, Label Free) provides only raw spectra without a pre-computed quantitative matrix; PDC000116 (CPTAC Prospective, 102 patients, TMT) is an independent cohort with zero TCGA patient overlap. The proteomic feature channel is therefore zero-filled for all 690 patients. A proteomics ablation in the companion MethodsX article confirms negligible impact (ΔF1 = +0.0016).

3. **Non-GPR reaction bias.** 4,663 of 10,600 reactions (44%) lack GPR rules and receive a default "active" label, inflating the active class. Use `gpr_reaction_mask.pt` to restrict analyses to the 5,937 GPR-mapped reactions.

4. **Subsystem coverage.** 3,961 reactions (37.4%) could not be mapped to metabolic subsystems via the Human-GEM BiGG cross-reference. These are predominantly transport and exchange reactions. They participate in training but are excluded from subsystem-level analysis.

5. **Gene symbol collisions.** HGNC symbols with duplicate Ensembl mappings are resolved by retaining the first occurrence. Users requiring Ensembl-level resolution should remap from raw GDC files.

---

## Recommended Evaluation Protocol

1. Use the 220-patient subset (`evaluation_subset_ids.json`) with expression-thresholded labels for comparability with the companion MethodsX article (AUROC 0.861, F1 0.796).
2. Report both all-reaction (10,600) and GPR-only (5,937) metrics via `gpr_reaction_mask.pt`.
3. Use AUROC as the primary threshold-independent metric, supplemented by F1, precision, and recall.
4. Recompute consensus pseudo-labels from training-split patients only to avoid transductive label leakage.
5. Specify the shared_metabolite sparsification parameter k for reproducibility.

---

## Reproducibility

- All data files include SHA-256 checksums in `MANIFEST.sha256`
- Validation script: `code/05_validate_dataset.py`
- All dependency versions pinned in `environment.yml`
- Archived on Zenodo with DOI for long-term persistence

---

## Ethics

All data are de-identified and accessed via standard TCGA data use agreements (dbGaP phs000178). No new patient samples were collected. No additional institutional ethical approval was required.

---

## Licence

- **Dataset:** Creative Commons Attribution 4.0 International (CC-BY 4.0)
- **Code:** MIT Licence
