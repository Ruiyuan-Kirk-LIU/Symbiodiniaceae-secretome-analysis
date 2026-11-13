# Symbiodiniaceae secretome analysis

Analysis code for a comparative secretome study of seven Symbiodiniaceae strains (six *Symbiodinium* species and *Breviolum minutum*): functional annotation, orthogroup classification, Pfam domain enrichment, secretion-evolution gene trees, exoproteome overlap, and generation of all figures and tables in the manuscript.

> **Manuscript:**  *Comparative secretomics reveals conserved enzymatic repertoires with divergent secretion in Symbiodiniaceae.* Liu, Y. *et al.* (under review).
>
> **Archived version of this repository:** doi:[10.5281/zenodo.24098913.](https://doi.org/10.5281/zenodo.24098913)

Secreted proteins were predicted with a separate, reusable pipeline:
Liu, Y. (2026). **Secretome Prediction Pipeline (Version v0.1.1)** [Computer software]. *Zenodo*. https://doi.org/10.5281/zenodo.20712666.

This repository starts from its outputs.

---

## Contents

- [Strains](#strains)
- [Repository structure](#repository-structure)
- [Requirements](#requirements)
- [Data](#data)
- [How to reproduce the analyses](#how-to-reproduce-the-analyses)
- [Figure and table map](#figure-and-table-map)
- [Software versions](#software-versions)
- [Citation](#citation)
- [Licence](#licence)

---

## Strains

Strains are always processed and plotted in this phylogenetic order. *B. minutum* (clade B) is separated from the *Symbiodinium* strains (clade A) in all figures.

| Key | Species | Clade | Proteome source | Assembly accession |
|---|---|---|---|---|
| `Spil` | *Symbiodinium pilosum* | A | JGI PhycoCosm | GCA_905231905.1 (Spil_CCMP2461)  |
| `Snat` | *Symbiodinium natans* | A | NCBI Genome | GCA_905221605.1 (Snat_CCMP2548) |
| `Stri` | *Symbiodinium tridacnidorum* | A | NCBI Genome | GCA_905221615.1 (Stri_CCMP2592) |
| `Slin` | *Symbiodinium linucheae* | A | NCBI Genome | GCA_905221635.1 (Slin_CCMP2456) |
| `Snec` | *Symbiodinium necroappetens* | A | NCBI Genome | GCA_905231915.1 (Snec_CCMP2469) |
| `Smic` | *Symbiodinium microadriaticum* | A | NCBI Genome | GCA_001939145.1 (ASM193914v1) |
| `Bmin` | *Breviolum minutum* | B | NCBI Genome | GCA_000507305.1 (ASM50730v1) |

Strain keys, plotting order, and colours are defined once in [`config/strains.tsv`](config/strains.tsv) and read by every script and notebook.

---

## Repository structure

```
symbiodiniaceae-secretome-analysis/
├── README.md
├── LICENSE
├── CITATION.cff
├── envs/                          # conda environments (one per compute context)
│   ├── annotation.yml             # HPC: eggNOG-mapper, InterProScan, Foldseek, dbCAN, EffectorP
│   ├── orthology.yml              # HPC: OrthoFinder + MMseqs2 (GPU)
│   └── analysis.yml               # Colab / local: pandas, scipy, statsmodels, matplotlib …
├── config/
│   ├── config.yaml                # root paths (relative to the repository)
│   └── strains.tsv                # strain keys, order, clade, colours
├── 01_secretome_prediction/       # how pipeline outputs were generated and collected
├── 02_orthology/                  # OrthoFinder inference; orthogroup secretome tagging
│                                  # (Secretome only / Non-secretome only / Mixed)
├── 03_functional_annotation/      # eggNOG-mapper + InterProScan + Foldseek union → GO_final / Pfam_final;
│                                  # dbCAN and MEROPS classification
├── 04_pfam_enrichment/            # Fisher's exact tests with BH correction, pooled and per strain
├── 05_exoproteome_overlap/        # prediction vs. LC-MS/MS detection overlap (Smic, Snat, Bmin)
├── 06_secretion_evolution/        # gene trees and parsimony reconstruction of signal-peptide gain/loss
├── 07_figures_tables/             # one notebook or script per main/supplementary figure and table
└── data/
    ├── README.md                  # where the large input and intermediate files are archived
    └── processed/                 # small tables sufficient to redraw every figure
```

Inside each numbered folder:

- `hpc/` – PBS Pro job scripts (run on the cluster)
- `scripts/` – Python/R scripts
- `notebooks/` – Jupyter/Colab notebooks

---

## Requirements

| Context | Used for | Environment |
|---|---|---|
| HPC cluster (PBS Pro, NVIDIA A100) | annotation, OrthoFinder, Foldseek | `envs/annotation.yml`, `envs/orthology.yml` |
| Google Colab or local machine (tested on Apple M2) | enrichment, overlap, gene-tree analyses, figures | `envs/analysis.yml` |

```bash
conda env create -f envs/analysis.yml
conda activate secretome-analysis
```

The PBS scripts assume a PBS Pro scheduler. Edit the resource lines (`#PBS -l …`) and module/conda activation lines for your cluster.

Several tools need separate licences or databases (eggNOG, InterProScan, Foldseek, dbCAN and MEROPS databases). See each folder's `README.md`.

---

## Data

Large files are **not** stored in this repository:

| Data | Location |
|---|---|
| Input proteomes (7 strains) | See [`data/README.md`](data/README.md) |
| LC-MS/MS exoproteome raw data | <!-- TODO: PRIDE / jPOST accession --> |

`data/processed/` contains the small summary tables needed to regenerate every figure without rerunning the HPC steps. Key files:

| File | Content |
|---|---|
| `orthogroup_secretome_tagging_wide.tsv` | per-orthogroup SP+ / SP− / not_secreted tags per strain (semicolon-delimited) |
| `ALL_whole_proteome_Pfam_final.tsv` | Pfam domains per protein (comma-delimited) |
| `pfam_enrichment_secretome_vs_background.tsv` | enrichment statistics per Pfam domain |
| <!-- TODO: add others --> | |

---

## How to reproduce the analyses

1. **Download** the archived data into `data/` (see [`data/README.md`](data/README.md)) and set `root` in `config/config.yaml`.
2. **Predict secretomes** with [secretome_pipeline_repo](https://github.com/Ruiyuan-Kirk-LIU/secretome_pipeline_repo) (CD-HIT → SignalP 6.0 → DeepLoc 2.1 → DeepTMHMM → TargetP 2.0 → ASAFind 2.0 → ER-retention filter), or use the archived secretome FASTA files. Collect them with `01_secretome_prediction/`.
3. **Infer orthogroups** with OrthoFinder (`02_orthology/hpc/`), then tag them by secretion status (`03_orthology/scripts/`). The manuscript uses the MCL-level orthogroups in `Orthogroups.tsv`.
4. **Annotate** whole proteomes on the HPC (`03_functional_annotation/hpc/`). Then merge eggNOG-mapper, InterProScan and Foldseek outputs into `GO_final` / `Pfam_final` (`03_functional_annotation/scripts/`).
5. **Test Pfam enrichment** in secretome vs. non-secretome proteins (`04_pfam_enrichment/`).
6. **Compare** predictions with exoproteome detections (`05_exoproteome_overlap/`).
7. **Reconstruct secretion evolution** on gene trees of selected orthogroups (`06_secretion_evolution/`).
8. **Generate figures and tables** (`07_figures_tables/`). Figures are written as PDF, SVG and PNG (600 dpi).

Steps 5–8 run from `data/processed/` alone.

---

## Figure and table map


| Item | Content | Script / notebook | Main input |
|---|---|---|---|
| Fig. 1  | Proteome sizes, completeness and predicted secretome sizes (SP+ / SP−) | `07_figures_tables/fig1_…` | proteomes, secretome FASTA files |
| Fig. S1 | | `07_figures_tables/figS1_…` | |
| Fig. 2  | Orthogroups and secretion status | `07_figures_tables/fig2_…` | `orthogroup_secretome_tagging_wide.tsv` |
| Fig. S2 | | `07_figures_tables/figS2_…` | |
| Fig. 3  | Pfam domain enrichment | `07_figures_tables/fig3_…` | `pfam_enrichment_secretome_vs_background.tsv` |
| Fig. S3 | | `07_figures_tables/figS3_…` | |
| Fig. 4  | Functional classification (CAZymes and peptidases) | `07_figures_tables/fig4_…` | annotation tables |
| Fig. 5  | Prediction and detection overlaps | `07_figures_tables/fig5_…` | secretome FASTA files, LC-MS/MS raw data, annotation tables |
| Fig. S5 | | `07_figures_tables/figS5_…` | |
| Fig. 6  | Secretion evolution of representative enzyme orthogroups | `07_figures_tables/fig6_…` | orthogroup, gene trees |
| Fig. S6 | | `07_figures_tables/figS6_…` | |
| Fig. 7  | Summary schematic | — (drawn manually) | — |
| Table S1 | Candidate orthogroups | `07_figures_tables/table1_…` | |
| Table S2 | Secretome-associated orthogroups | `07_figures_tables/tableS1_…` | |

---

## Software versions

<!-- TODO: must match Table 2 of the Methods exactly -->

| Tool | Version | Step | Reference |
|---|---|---|---|
| CD-HIT | 4.8.1 | redundancy removal | Fu *et al.* (2012) *Bioinformatics* 28:3150–3152 |
| SignalP | 6.0 | signal peptides | Teufel *et al.* (2022) *Nat Biotechnol* 40:1023–1025 |
| DeepLoc | 2.1 | localisation | Ødum *et al.* (2024) *Nucleic Acids Res* 52:W215–W220 |
| DeepTMHMM | 1.0 | transmembrane helices | Hallgren *et al.* (2022) bioRxiv 2022.04.08.487609 |
| TargetP | 2.0 | organelle targeting | Almagro Armenteros *et al.* (2019) *Life Sci Alliance* 2:e201900429 |
| ASAFind | 2.0 | plastid targeting | Gruber *et al.* (2015) *Plant J* 81:519–528 |
| NetGPI  | 1.1 | GPI-anchored | Gíslason *et al.* (2015) *Curr Res Biotechnol* 3:6-13 |
| OrthoFinder | 3.1.2 | orthogroups | Emms *et al.* (2019) *Genome Biol* 20:238 |
| MMseqs2 |  | sequence search for OrthoFinder | Steinegger *et al.* (2017) *Nat Biotechnol* 35:1026-1028 |
| DendroBLAST | | gene tree for OrthoFinder | Kelly *et al.* (2013) *PLoS ONE* 8:e58537 |
| eggNOG-mapper | 2.1.12 | annotation | Cantalapiedra *et al.* (2021) *Mol Biol Evol* 38:5825–5829 |
| InterProScan | 5.75-106.0 | annotation | Blumn *et al.* (2025) *Nucleic Acids Res* 54:D444-D456 |
| Foldseek | 10.941cd33 | structure-based annotation (`afdb_swissprot`) | van Kempen *et al.* (2024) *Nat Biotechnol* 42:243–246 |
| dbCAN | 5 | CAZymes | Zheng *et al.* (2023) *Nucleic Acids Res* 51:W115-W121 |
| MEROPS | | peptidases | Rawlings *et al.* (2018) *Nucleic Acids Res* 46:D624–D632 |
| IQ-TREE | 3 | secretion evolution | Wong *et al.* (2026) *Mol Biol Evol* 43:msag117 |
| Python | 3.12 | analyses and figures | |

---

## Citation

If you use this code, please cite the paper and the archived release:

```
Liu, Y. et al. (2026) Symbiodiniaceae secretome analysis. Zenodo. Available at: https://doi.org/10.5281/zenodo.24098913.
```

See also [`CITATION.cff`](CITATION.cff).

## Licence

Released under the [MIT Licence](LICENSE).

## Contact

Yuxin Liu, The University of Tokyo – [Email](ruiyuanliu0723@gmail.com).
