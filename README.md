# PB&MB Bulk RNA-seq Analysis
DESeq2-based exploratory bulk RNA-seq analysis of PB, RB and MB patient samples.
## Project Aim
This project explores transcriptomic similarities and differences between pineoblastoma(PB) and medulloblastoma(MB) Group 3.
## Current Analysis Strategy
Currently, publicly available St. Jude HTSeq feature counts datasets from PB, RB, and MB cohorts are being integrated for exploratory transcriptomic analysis. Because explicit MB Group 3 and Group 4 annotations are not consistently available in the current merged dataset, the current main focus on the PB vs MB_nonWNT_nonSHH, which is currently used as the closest available proxy for MB Group 3/4.
## Overall Project Plan
### Step 1. same-platform exploratory comparison
Compare PB with MB_nonWNT_nonSHH from the same St. Jude platform.
### Step 2. Subgroup-refined comparison
Use external MB datasets with explicit subgroup annotation.
## Repository Structure
```text
PB_MB_bulkseq_analysis/
├── README.md
├── scripts/
│   ├── 01_data_loading_and_metadata.R
│   ├── 02_count_matrix_integration.R
│   ├── 03_qc_and_pca.R
│   └── 04_deseq2_pb_vs_mb_nonWNT_nonSHH.R
├── results/
│   ├── figures/
│   └── tables/
└── notes/
```
### Script Overview

| Script | Purpose |
|---|---|
| 01_data_loading_and_metadata.R | Load PB/RB/MB datasets and define analysis groups |
| 02_count_matrix_integration.R | Merge count matrices using common gene symbols |
| 03_qc_and_pca.R | Perform QC, outlier removal, VST transformation, and PCA |
| 04_deseq2_pb_vs_mb_nonWNT_nonSHH.R | Run DESeq2 differential expression analysis |

## Workflow Overview
1. **Data Loading:** Load PB, RB, and MB HTSeq raw count matrices and sample metadata  

2. **Metadata Intergration:** Define analysis groups and merge metadata  

3. **Count Matrix Intergration:** Merge count matrices by common gene symbols  

4. **QC and PCA:** Perform QC and PCA  

5. **Differential Expression:** Run DESeq2 analysis for PB vs MB_nonWNT_nonSHH  

6. **Biological Interpretation:** Interpret results at pathway/program level
## Detailed Script Descriptions
### 01_data_loading_and_metadata.R
This script loads PB, RB, and MB count matrices and sample metadata, assigns analysis groups, merges metadata tables, and exports the initial metadata and group summary.
#### Outputs
- `results/tables/PB_RB_MB_merged_metadata.csv`
- `results/tables/group_summary_initial.csv`
#### Group_summary_initial
| Group | Sample count |
|---|---:|
| PB | 23 |
| RB | 98 |
| MB_WNT | 36 |
| MB_SHH | 91 |
| MB_nonWNT_nonSHH | 192 |
| Total | 440 |

### 02_count_matrix_integration.R
This script loads PB, RB, and MB raw count matrices, identifies common gene symbols across datasets, merges count matrices into a unified raw count matrix, and exports integration summary tables.
#### Outputs
The merged count matrix contains 59,050 common genes and 440 samples.  
The additional first column stores gene symbols, resulting in 441 total columns in the exported CSV file.
- `results/tables/PB_RB_MB_merged_counts_raw.csv`
- `results/tables/count_matrix_integration_summary.csv`
#### Count matrix integration summary
| Item | Value |
|---|---:|
| PB genes | 59055 |
| RB genes | 59055 |
| MB genes | 59050 |
| Common genes | 59050 |
| Merged count matrix rows | 59050 |
| Merged count matrix columns | 441 |
| Merged samples | 440 |
| Metadata samples | 440 |
| Samples matched | TRUE |

### 03_qc_and_pca.R

This script converts the merged raw count matrix into a DESeq2-compatible matrix, performs basic sample-level QC, removes one low-depth technical outlier, applies VST transformation, and generates PCA plots.

#### QC / data cleaning decision
During initial QC, one sample (`SJRB030317_D1`) showed extremely low library size (~0.4M reads) compared with the cohort median (~24M) and was removed as a likely technical outlier.

| QC step | Sample count | Minimum library size | 1st quartile | Median library size | Mean library size | 3rd quartile | Maximum library size |
|---|---:|---:|---:|---:|---:|---:|---:|
| Before outlier removal | 440 | 436,256 | 16,324,791 | 24,604,978 | 27,250,300 | 35,865,518 | 99,014,426 |
| After outlier removal | 439 | 1,071,437 | 16,431,300 | 24,625,994 | 27,311,380 | 35,883,158 | 99,014,426 |

Only one sample was removed during QC, and the overall library size distribution remained stable after filtering, suggesting that the exclusion primarily affected an extreme low-depth outlier rather than the global cohort structure.
#### Final Analysis-ready dataset
| Group | Sample count |
|---|---:|
| PB | 23 |
| RB | 97 |
| MB_WNT | 36 |
| MB_SHH | 91 |
| MB_nonWNT_nonSHH | 192 |
| Total | 439 |

#### Outputs
- `results/tables/PB_RB_MB_counts_after_QC.csv`
- `results/tables/PB_RB_MB_metadata_after_QC.csv`
- `results/tables/qc_summary.csv`
- `results/figures/PCA_analysis_group_after_QC.png`
- `results/figures/PCA_sequencing_platform_after_QC.png`
- `results/figures/PCA_tissue_preservative_after_QC.png`
- `results/figures/PCA_dataset_after_QC.png`
- `results/figures/PCA_QC_overview_after_QC.png`

#### PCA overview
VST-based PCA was used to evaluate sample-level structure after QC. PCA plots were colored by analysis group, sequencing platform, tissue preservation method, and dataset source.

The PCA showed clear biological separation by analysis group. Additional PCA plots colored by technical variables showed no strong global separation by sequencing platform, tissue preservation method, or dataset source, suggesting that technical batch effects are unlikely to be the dominant driver of the overall expression structure.

![PCA QC overview](results/figures/PCA_QC_overview_after_QC.png)

### 04_deseq2_pb_vs_mb_nonWNT_nonSHH.R

This script performs differential expression analysis between PB and MB_nonWNT_nonSHH using QC-filtered raw counts. DESeq2 was used with `analysis_group` as the design variable. `MB_nonWNT_nonSHH` was set as the reference group, so positive log2 fold changes indicate higher expression in PB.

#### Comparison

| Group | Sample count |
|---|---:|
| PB | 23 |
| MB_nonWNT_nonSHH | 192 |
| Total | 215 |

#### DESeq2 summary

| Category | Gene count |
|---|---:|
| Up in PB | 9,172 |
| Up in MB_nonWNT_nonSHH | 7,437 |
| Not significant | 32,540 |
| Not tested / low-count filtered | 7,719 |

DESeq2 identified broad transcriptional differences between PB and MB_nonWNT_nonSHH. These gene-level results will be interpreted further at the pathway and gene-program level rather than relying only on individual top-ranked genes.

#### Outputs

- `results/deseq2/DESeq2_PB_vs_MB_nonWNT_nonSHH.csv`
- `results/deseq2/DESeq2_PB_vs_MB_nonWNT_nonSHH_normalized_counts.csv`
- `results/tables/DESeq2_PB_vs_MB_nonWNT_nonSHH_summary.csv`

#### Volcano plot

The volcano plot provides an overview of gene-level differential expression between PB and MB_nonWNT_nonSHH. It is used as a summary visualization, while biological interpretation will be performed at the pathway and gene-program level.

![Volcano plot](results/figures/Volcano_PB_vs_MB_nonWNT_nonSHH.png)

