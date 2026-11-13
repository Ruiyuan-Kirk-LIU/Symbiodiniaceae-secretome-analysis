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
│   ├── annotation.yml             # HPC: eggNOG-mapper, InterProScan, Foldseek, dbCAN
│   ├── orthology.yml              # HPC: OrthoFinder + MMseqs2 (GPU)
│   └── analysis.yml               # Colab / local: pandas, scipy, statsmodels, matplotlib …
├── config/
│   ├── config.yaml                # root paths (relative to the repository)
│   └── strains.tsv                # strain keys, order, clade, colours
├── 01_secretome_prediction/       # pipeline outputs per strain (01_cdhit … 09_final, summary/);
│   └── Result1_basic_analysis_of_predicted_secretome.ipynb
├── 02_orthology/                  # OrthoFinder results; orthogroup secretome tagging
│   │                              # (Secretome only / Non-secretome only / Mixed)
│   └── Result2_Orthogroups.ipynb
├── 03_functional_annotation/      # Pfam/GO annotation of secretomes, Pfam enrichment;
│   │                              # dbCAN (dbcan_out/) and MEROPS (merops_out/) results
│   ├── Result3_Secretome_annotations.ipynb
│   └── Result4_database_functional_classes.ipynb
├── 04_pfam_enrichment/            # per-strain Pfam maps and Fisher's exact test results (output of Result3)
├── 05_exoproteome_overlap/        # LC-MS/MS protein reports (Smic, Snat, Bmin) and prediction–detection overlap
│   └── Result5_Prediction_vs_Detection.ipynb
├── 06_secretion_evolution/        # gene trees and parsimony reconstruction of signal-peptide gain/loss
│   └── Result6_Secretion_evolution.ipynb
├── 07_figures_tables/             # final figures, one folder per main/supplementary figure
│                                  # (fig1_… to fig7_…, figS1_… to figS8_…), each as PDF/SVG/PNG
└── data/
    ├── README.md                  # contents of data/processed and where external data are archived
    └── processed/                 # analysis-ready tables read by the notebooks
```

Inside each numbered folder:
Each analysis step has one Jupyter/Colab notebook (`Result1` to `Result6`), stored in the folder of that step together with its outputs. The figures each notebook writes are collected in `07_figures_tables/` (see the [figure and table map](#figure-and-table-map)).

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
| Input proteomes (7 strains) | See [`data/README.md`](data/README.md), accessions in [Strains](#Strains) |
| Functional annotation of Symbiodiniaceae proteome | Liu, Y. et al. (2026) “Functional annotation of Symbiodiniaceae proteome”. Zenodo. Available at: [https://doi.org/10.5281/zenodo.24190496](https://doi.org/10.5281/zenodo.24190496). |


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
2. **Predict secretomes** with [secretome_pipeline_repo](https://github.com/Ruiyuan-Kirk-LIU/secretome_pipeline_repo) (CD-HIT → SignalP 6.0 → DeepLoc 2.1 → DeepTMHMM → TargetP 2.0 → ASAFind 2.0 → ER-retention filter → NetGPI), or use the outputs in `01_secretome_prediction/<strain>/09_final/`. Summarise them with `Result1_basic_analysis_of_predicted_secretome.ipynb`.
3. **Infer orthogroups** with OrthoFinder (results in `02_orthology/`), then tag them by secretion status with `Result2_Orthogroups.ipynb`. The manuscript uses the MCL-level orthogroups in `Orthogroups/Orthogroups.tsv`.
4. **Annotate** whole proteomes (eggNOG-mapper, InterProScan, Foldseek; archived on Zenodo, see [Data](#data)), merge them into `GO_final` / `Pfam_final`, and **test Pfam enrichment** in secretome vs. non-secretome proteins with `03_functional_annotation/Result3_Secretome_annotations.ipynb`. Enrichment tables are written to `04_pfam_enrichment/`.
5. **Classify CAZymes and peptidases** (dbCAN, MEROPS) with `03_functional_annotation/Result4_database_functional_classes.ipynb`.
6. **Compare** predictions with exoproteome detections with `05_exoproteome_overlap/Result5_Prediction_vs_Detection.ipynb`.
7. **Reconstruct secretion evolution** on gene trees of selected orthogroups with `06_secretion_evolution/Result6_Secretion_evolution.ipynb`.
Each notebook saves its figures as PDF, SVG and PNG (600 dpi). The final versions are collected in `07_figures_tables/`.


---

## Figure and table map

| Item | Content | Notebook | Figure folder (`07_figures_tables/`) | Main input |
|---|---|---|---|---|
| Fig. 1  | Summary statistics of proteomes and secretomes | `Result1` | `fig1_proteome_secretome_size/` | proteomes, `01_secretome_prediction/*/09_final/` |
| Fig. 2  | Orthogroup clustering of secreted and non-secreted proteins | `Result2` | `fig2_orthogroup/` | `orthogroup_secretome_tagging_wide.tsv` |
| Fig. 3  | Pfam enrichment of the secretome and its distribution across strains | `Result3` | `fig3_secretome_enriched_Pfam/` | `ALL_whole_proteome_Pfam_final.tsv`, `04_pfam_enrichment/` |
| Fig. 4  | CAZyme and peptidase content of predicted secretomes | `Result4` | `fig4_CAZyme_and_peptidase/` | `03_functional_annotation/dbcan_out/`, `merops_out/` |
| Fig. 5  | Pfam-based annotation of proteins shared between MS detection and prediction | `Result5` | `fig5_prediction_and_detection/` | `05_exoproteome_overlap/*_Proteins.txt`, secretomes, Pfam table |
| Fig. 6  | Gains and losses of predicted secretion in the enzyme orthogroups | `Result6` | `fig6_secretion_gain_and_loss/` | orthogroups, gene trees, `06_secretion_evolution/*.tsv` |
| Fig. 7  | Summary schematic | — (drawn manually) | `fig7_summary/` | — |
| Fig. S1 | Physicochemical characteristics of the predicted secretomes | `Result1` | `figS1_physicochemical_characteristics/` | secretome FASTA files |
| Fig. S2 | Orthogroup-level clustering and strain-level similarity | `Result2` | `figS2_orthogroup_presence_patterns/` | `orthogroup_secretome_tagging_wide.tsv` |
| Fig. S3 | Secretome-enriched Pfam domains across the seven strains | `Result3` | `figS3_secretome_enriched_Pfam/` | `04_pfam_enrichment/` |
| Fig. S4 | CAZyme family repertoire of predicted secretomes | `Result4` | `figS4_CAZyme/` | `03_functional_annotation/dbcan_out/` |
| Fig. S5 | Peptidase family repertoire of predicted secretomes | `Result4` | `figS5_peptidase/` | `03_functional_annotation/merops_out/` |
| Fig. S6 | Functional composition of MS-detected proteins | `Result5` | `figS6_functional_composition_detect/` | `05_exoproteome_overlap/*_Proteins.txt` |
| Fig. S7 | Gene trees of 23 supplementary orthogroups carrying enriched enzyme domains | `Result6` | `figS7_gene_tree_23_supple/` | orthogroups, gene trees |
| Fig. S8 | Identification-confidence diagnostics for the LC-MS/MS datasets | `Result5` | `figS8_detection_diagnostics/` | `05_exoproteome_overlap/*_Proteins.txt` |
| Table S1 | Candidate orthogroups | <!-- TODO --> | — | |
| Table S2 | Secretome-associated orthogroups | <!-- TODO --> | — | `orthogroup_secretome_tagging_wide.tsv` |
 
Notebook names are abbreviated: `Result1` = `01_secretome_prediction/Result1_basic_analysis_of_predicted_secretome.ipynb`, `Result2` = `02_orthology/Result2_Orthogroups.ipynb`, `Result3` / `Result4` = `03_functional_annotation/Result3_Secretome_annotations.ipynb` / `Result4_database_functional_classes.ipynb`, `Result5` = `05_exoproteome_overlap/Result5_Prediction_vs_Detection.ipynb`, `Result6` = `06_secretion_evolution/Result6_Secretion_evolution.ipynb`.
 

---

## Software versions

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
