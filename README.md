# ProdiRank

**Comparator-Informed Computational Prioritization of Prodiginine Anti-Cancer Analogues**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/)
[![RDKit](https://img.shields.io/badge/RDKit-2023.03-green.svg)](https://www.rdkit.org/)

> AR INBRE Pilot Project | UAMS × UALR 
> **Mentor/PI:** Kalyani Dhusia, Ph.D. · UAMS Department of Physiology & Cell Biology · AR INBRE Data Science Core  
> **Project Leader:** Brian Walker, Ph.D. · UALR Department of Chemistry  
> **Collaborator:** Robert Griffin, Ph.D. · UAMS Radiation Oncology · Winthrop P. Rockefeller Cancer Institute

---

## Overview

ProdiRank is a computational decision framework that integrates three independent evidence streams into ranked synthesis/testing recommendations for PUI medicinal chemists:

| Stream | Source | Key output |
|---|---|---|
| Compound/activity data | Walker laboratory (UALR) | 13 SMILES-validated prodiginine analogues; BW1 dimer CCK-8 EC50 |
| Cancer pharmacogenomics | GDSC1 (651 cell lines) | BCL2L2 r=0.190 as top obatoclax sensitivity correlate |
| Structure-informed cheminformatics | PDB, AlphaFold, RDKit | MCDA-ranked synthesis/testing decision matrix |

### Key finding

Genome-wide Spearman correlation of obatoclax (GX15-070) sensitivity across **651 GDSC1 cancer cell lines** shows that **BCL2** (obatoclax's declared target) is **not a significant predictor** (r = −0.023, p = 0.699), while **BCL2L2** (r = 0.190, FDR = 3.4×10⁻⁵) and **IQSEC2** (r = 0.212, FDR = 4×10⁻⁶) are the dominant empirical sensitivity correlates — demonstrating that systematic off-target profiling before synthesis is the missing step in prodiginine optimization.

---

## Repository structure

```
ProdiRank/
├── data/
│   └── raw/                        # Raw input files (SMILES, CCK-8 plates)
├── environment/
│   ├── prodirank_py.yml            # Python conda environment
│   └── prodirank_r.yml             # R conda environment
├── notebooks/
│   ├── 01_data_intake_cleaning.ipynb       # Compound manifest + CCK-8 parsing
│   ├── 02_obatoclax_pharmacogenomics.ipynb # GDSC1 genome-wide correlation
│   ├── 03_wp1_tanimoto_umap.ipynb          # Tanimoto similarity + UMAP
│   ├── 04_wp1_admet_screening.ipynb        # Lipinski/PAINS/Brenk screening
│   └── 05_activity_cck8_parsing.ipynb      # EC50 estimation from CCK-8 plates
├── outputs/
│   ├── figures/                    # All publication-ready figures (code-generated)
│   └── tables/                     # CSV/XLSX output tables
├── scripts/
│   ├── clean_compound_manifest.py
│   ├── parse_cck8_activity.py
│   ├── run_admet_screen.py
│   └── run_tanimoto_umap.py
├── .gitignore
├── LICENSE
└── README.md
```

---

## Quick start

```bash
# 1. Clone
git clone https://github.com/kalyanidhusia/ProdiRank.git
cd ProdiRank

# 2. Create environment
conda env create -f environment/prodirank_py.yml
conda activate prodirank

# 3. Launch notebooks
jupyter lab notebooks/
```

Run notebooks in order (01 → 05). Each notebook is self-contained and generates outputs to `outputs/figures/` and `outputs/tables/`.

---

## Notebooks

| # | Notebook | What it does | Key output |
|---|---|---|---|
| 01 | `01_data_intake_cleaning.ipynb` | SMILES validation (RDKit), compound manifest, CCK-8 plate parsing | `compound_manifest_clean.csv`, `cck8_summary_tidy.csv` |
| 02 | `02_obatoclax_pharmacogenomics.ipynb` | Genome-wide Spearman correlation (GDSC1), volcano plot, BCL2L2 discovery | `public_obatoclax_gene_summary.csv`, `fig8_grant_multipanel.png` |
| 03 | `03_wp1_tanimoto_umap.ipynb` | Morgan ECFP4 fingerprints, pairwise Tanimoto, UMAP chemical space | `wp1_tanimoto_umap.png` |
| 04 | `04_wp1_admet_screening.ipynb` | Lipinski Ro5, Veber, Ghose, PAINS A/B/C, Brenk alerts, QED | `wp1_admet_trafficlight.png` |
| 05 | `05_activity_cck8_parsing.ipynb` | 4-parameter logistic EC50 fitting, dose-response curves | `ec50_summary.csv` |

---

## Key results (preliminary data)

**Compound characterization:**
- 13/13 SMILES validated by RDKit MolStandardize
- Tanimoto similarity to obatoclax: **0.09–0.22** (genuinely distinct chemotype, not me-too)
- All 13 compounds satisfy Lipinski Ro5; no PAINS A/B/C alerts detected

**GDSC1 pharmacogenomics (651 cell lines):**
- BCL2L2: r = **0.190**, FDR = 3.4×10⁻⁵ ← **primary docking target**
- IQSEC2: r = **0.212**, FDR = 4×10⁻⁶
- BCL2 (declared target): r = **−0.023**, p = 0.699 (not significant)

**CCK-8 activity:**
- BW1 dimer EC50: **1.2–2.9 µM** across murine endothelial models
- Potent range comparable to obatoclax in responsive lines

---

## ProdiRank scoring framework

ProdiRank implements a five-dimension weighted multi-criteria decision analysis (MCDA):

| Evidence dimension | Weight | Input | Source |
|---|---|---|---|
| Chemical similarity to active comparators | 20% | Tanimoto (Morgan ECFP4) | Walker SMILES, ChEMBL |
| Pharmacogenomics support score | 25% | BCL2L2/IQSEC2 Spearman r | GDSC1, CTRP |
| Structural plausibility | 25% | AutoDock Vina score; fpocket | PDB, AlphaFold DB |
| ADMET / liability profile | 20% | QED; PAINS/Brenk; Lipinski | RDKit |
| Synthesis feasibility | 10% | Confirmed synthesis status | Walker laboratory |

Missing data → proportional weight redistribution.  
Output categories: **Prioritize / Monitor / Deprioritize / Needs Clarification**

---

## Figures (all code-generated, no AI)

| File | Description | Generated by |
|---|---|---|
| `wp1_tanimoto_umap.png` | Chemical space UMAP — Walker analogues vs. obatoclax | `notebooks/03` |
| `wp1_admet_trafficlight.png` | ADMET traffic-light heatmap for all 13 compounds | `notebooks/04` |
| `fig8_grant_multipanel.png` | GDSC1 pharmacogenomics multipanel (grant Figure 2) | `notebooks/02` |
| `fig2_volcano_plot.png` | Genome-wide volcano — BCL2L2 vs BCL2 | `notebooks/02` |
| `fig3_bcl2_family_scatter.png` | BCL2 family scatter plots (all weak correlates) | `notebooks/02` |
| `fig4_top_correlates.png` | SDC4, LMNA, IQSEC2 as top obatoclax correlates | `notebooks/02` |
| `obatoclax_gene_correlation_summary.png` | Ranked gene correlation summary | `notebooks/02` |

---

## Software environment

| Package | Version | Purpose |
|---|---|---|
| Python | 3.9+ | Core language |
| RDKit | 2023.03 | SMILES validation, descriptors, fingerprints |
| pandas / numpy | latest | Data manipulation |
| scipy | latest | Spearman correlation, 4PL fitting |
| matplotlib / seaborn | latest | Figures |
| umap-learn | 0.5+ | Chemical space UMAP |
| scikit-learn | latest | Random forest feature importance |

See `environment/prodirank_py.yml` for the complete locked environment.

---

## Data availability

- **Input SMILES and CCK-8 data:** Walker laboratory (OBU) — available upon request to project leaders
- **GDSC1 pharmacogenomics:** Publicly available at [depmap.org](https://depmap.org/portal/download/all/)
- **PRIDE proteomics datasets:** [ebi.ac.uk/pride](https://www.ebi.ac.uk/pride/)
- **ChEMBL benchmark:** [ebi.ac.uk/chembl](https://www.ebi.ac.uk/chembl/)

---

## Citation

If you use ProdiRank in your research, please cite:

```
Dhusia K, Walker B, Griffin RJ (2026). ProdiRank: Comparator-Informed Computational 
Prioritization of Prodiginine Anti-Cancer Analogues. AR INBRE Pilot Grant.
GitHub: https://github.com/kalyanidhusia/ProdiRank
```

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Acknowledgements

This work is supported by the Arkansas IDeA Network of Biomedical Research Excellence (AR INBRE), funded by the National Institute of General Medical Sciences (NIGMS) of the National Institutes of Health under Award Number P20GM103429. The GDSC1 data were generated by the Wellcome Sanger Institute. PRIDE is maintained by EMBL-EBI.
