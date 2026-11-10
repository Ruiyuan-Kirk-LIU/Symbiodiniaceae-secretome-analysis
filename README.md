# Symbiodiniaceae secretome analysis

Analysis code for a comparative secretome study of seven Symbiodiniaceae strains (six *Symbiodinium* species and *Breviolum minutum*): functional annotation, orthogroup classification, Pfam domain enrichment, secretion-evolution gene trees, exoproteome overlap, and generation of all figures and tables in the manuscript.

> **Manuscript:**  *Comparative secretomics reveals conserved enzymatic repertoires with divergent secretion in Symbiodiniaceae.* Liu R. *et al.* (under review).
>
> **Archived version of this repository:** <!-- Zenodo DOI of all versions after v0.0.1 release --> doi:10.5281/zenodo.24098913.

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
├── 02_functional_annotation/      # eggNOG-mapper + InterProScan + Foldseek union → GO_final / Pfam_final;
│                                  # dbCAN, MEROPS, EffectorP, lectin classification
├── 03_orthology/                  # OrthoFinder inference; orthogroup secretome tagging
│                                  # (Secretome only / Non-secretome only / Mixed)
├── 04_pfam_enrichment/            # Fisher's exact tests with BH correction, pooled and per strain
├── 05_secretion_evolution/        # gene trees and parsimony reconstruction of signal-peptide gain/loss
├── 06_exoproteome_overlap/        # prediction vs. LC-MS/MS detection overlap (Smic, Snat, Bmin)
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
| HPC cluster (PBS Pro, NVIDIA A100) | annotation, OrthoFinder/MMseqs2-GPU, Foldseek | `envs/annotation.yml`, `envs/orthology.yml` |
| Google Colab or local machine (tested on Apple M2) | enrichment, overlap, gene-tree analyses, figures | `envs/analysis.yml` |

```bash
conda env create -f envs/analysis.yml
conda activate secretome-analysis
```

The PBS scripts assume a PBS Pro scheduler. Edit the resource lines (`#PBS -l …`) and module/conda activation lines for your cluster.

Several tools need separate licences or databases (SignalP 6.0, TargetP 2.0 and DeepLoc 2.1 from DTU Health Tech; eggNOG, InterPro, AlphaFold/Swiss-Prot Foldseek and dbCAN databases). See each folder's `README.md`.

---

## Data

Large files are **not** stored in this repository:

| Data | Location |
|---|---|
| Input proteomes (7 strains) | <!-- TODO: original sources / accessions --> |
| Predicted secretomes (SP+ and SP−, FASTA) | <!-- TODO: Zenodo DOI --> |
| Annotation tables, OrthoFinder results | <!-- TODO: Zenodo DOI --> |
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
3. **Annotate** whole proteomes on the HPC (`02_functional_annotation/hpc/`). Then merge eggNOG-mapper, InterProScan and Foldseek outputs into `GO_final` / `Pfam_final` (`02_functional_annotation/scripts/`).
4. **Infer orthogroups** with OrthoFinder (`03_orthology/hpc/`), then tag them by secretion status (`03_orthology/scripts/`). The manuscript uses the MCL-level orthogroups in `Orthogroups.tsv`, not the N0 hierarchical orthogroups.
5. **Test Pfam enrichment** in secretome vs. non-secretome proteins (`04_pfam_enrichment/`).
6. **Reconstruct secretion evolution** on gene trees of selected orthogroups (`05_secretion_evolution/`).
7. **Compare** predictions with exoproteome detections (`06_exoproteome_overlap/`).
8. **Generate figures and tables** (`07_figures_tables/`). Figures are written as PDF, SVG and PNG (600 dpi).

Steps 5–8 run from `data/processed/` alone.

---

## Figure and table map

<!-- TODO: update numbers to the final manuscript -->

| Item | Content | Script / notebook | Main input |
|---|---|---|---|
| Fig. 1 | Secretome sizes (SP+ / SP−) | `07_figures_tables/fig1_…` | secretome FASTA files |
| Fig. 2 | Functional classification (CAZymes, peptidases, effector-like candidates, lectins) | `07_figures_tables/fig2_…` | annotation tables |
| Fig. 3 | Orthogroup membership and conservation | `07_figures_tables/fig3_…` | `orthogroup_secretome_tagging_wide.tsv` |
| Fig. 4 | Pfam domain enrichment | `07_figures_tables/fig4_…` | `pfam_enrichment_secretome_vs_background.tsv` |
| Fig. 5 | Secretion evolution of representative enzyme orthogroups | `07_figures_tables/fig5_…` | gene trees |
| Fig. 6 | Summary schematic | — (drawn manually) | — |
| Table 1 | Candidate orthogroups | `07_figures_tables/table1_…` | |
| Fig. S… | | | |
| Table S1 | Secretome-associated orthogroups | `07_figures_tables/tableS1_…` | |

---

## Software versions

<!-- TODO: must match Table 2 of the Methods exactly -->

| Tool | Version | Step | Reference |
|---|---|---|---|
| CD-HIT | <!-- TODO --> | redundancy removal | Fu *et al.* (2012) *Bioinformatics* 28:3150–3152 |
| SignalP | 6.0 | signal peptides | Teufel *et al.* (2022) *Nat Biotechnol* 40:1023–1025 |
| DeepLoc | 2.1 | localisation | Ødum *et al.* (2024) *Nucleic Acids Res* 52:W215–W220 |
| DeepTMHMM | <!-- TODO --> | transmembrane helices | Hallgren *et al.* (2022) bioRxiv 2022.04.08.487609 |
| TargetP | 2.0 | organelle targeting | Almagro Armenteros *et al.* (2019) *Life Sci Alliance* 2:e201900429 |
| ASAFind | 2.0 | plastid targeting | Gruber *et al.* (2015) *Plant J* 81:519–528 |
| eggNOG-mapper | 2.1.12 | annotation | Cantalapiedra *et al.* (2021) *Mol Biol Evol* 38:5825–5829 |
| InterProScan | 5.75-106.0 | annotation | Jones *et al.* (2014) *Bioinformatics* 30:1236–1240 |
| Foldseek | 10.941cd33 | structure-based annotation (`afdb_swissprot`) | van Kempen *et al.* (2024) *Nat Biotechnol* 42:243–246 |
| dbCAN | 5 | CAZymes | <!-- TODO --> |
| MEROPS | <!-- TODO: release --> | peptidases | Rawlings *et al.* (2018) *Nucleic Acids Res* 46:D624–D632 |
| EffectorP | 3.0 | effector-like candidates | Sperschneider & Dodds (2022) *Mol Plant Microbe Interact* 35:146–156 |
| OrthoFinder | 3.1.2 | orthogroups | <!-- TODO: citation for v3 --> |
| MMseqs2 (GPU) | <!-- TODO --> | sequence search for OrthoFinder | Kallenborn *et al.* (2025) *Nat Methods* <!-- TODO: check --> |
| <!-- TODO: alignment, tree and parsimony tools --> | | secretion evolution | |
| Python | <!-- TODO --> | analyses and figures | |

---

## Citation

If you use this code, please cite the paper and the archived release:

```
Liu R. et al. (YEAR) <title>. BMC Biology. doi:<!-- TODO -->
Liu R. (YEAR) symbiodiniaceae-secretome-analysis v1.0.0. Zenodo. doi:10.5281/zenodo.XXXXXXX
```

See also [`CITATION.cff`](CITATION.cff).

## Licence

Released under the [MIT Licence](LICENSE).

## Contact

Ruiyuan Liu, The University of Tokyo – <!-- TODO: email --> · issues and questions via [GitHub Issues](../../issues)
