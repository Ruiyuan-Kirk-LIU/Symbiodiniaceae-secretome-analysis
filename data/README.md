# Data

This folder holds the analysis-ready tables that the downstream notebooks read. Most figures in the manuscript can be redrawn from these files plus the small outputs kept in the numbered analysis folders. Nothing needs to be rerun on the HPC.

Large tool outputs stay with the step that produced them (`01_secretome_prediction/` to `06_secretion_evolution/`). The original input proteomes and the raw mass-spectrometry data are **not** stored in this repository. Their sources are listed below.

---

## `processed/`

| File | Rows | Produced by | Used by |
|---|---|---|---|
| `ALL_whole_proteome_Pfam_final.tsv` | 255,131 proteins | `03_functional_annotation/Result3_Secretome_annotations.ipynb` | Pfam enrichment, orthogroup domain summaries, Fig. 3 |
| `orthogroup_secretome_tagging_wide.tsv` | 30,844 orthogroups | `02_orthology/Result2_Orthogroups.ipynb` | orthogroup classification, Fig. 2, Fig. S2, Table S1 |
| `pfam_enrichment_secretome_vs_nonsecretome.tsv` | 44,141 tests | `04_pfam_enrichment/` | Fig. 3, Fig. S3 |

All files are tab-separated, UTF-8 encoded, and have one header row.

### `ALL_whole_proteome_Pfam_final.tsv`

Pfam domains for every protein in the seven whole proteomes. The domains are the union of eggNOG-mapper, InterProScan and Foldseek annotations.

| Column | Description |
|---|---|
| `strain` | Strain key (see [`config/strains.tsv`](../config/strains.tsv)); *S. microadriaticum* is `Smic_jgi` |
| `protein_id` | Protein identifier as in the source proteome (see [Identifier formats](#identifier-formats)) |
| `Pfam_final` | Pfam accessions, **comma-separated** (e.g. `PF00274,PF00583`); empty if no domain was assigned |

About 55% of proteins (141,236) have no Pfam domain. This mostly reflects how little of these lineages is covered by current databases, not missing annotation runs.

### `orthogroup_secretome_tagging_wide.tsv`

The secretion status of each OrthoFinder orthogroup in each strain. Orthogroups come from the MCL level in `Orthogroups.tsv` (30,844 orthogroups). The N0 hierarchical orthogroups (31,080) are not used.

| Column | Description |
|---|---|
| `orthogroup` | OrthoFinder orthogroup ID (`OG0000000` …) |
| `Spil` … `Bmin` | One column per strain, in phylogenetic order |

Cell values:

| Value | Meaning |
|---|---|
| `absent` | No member from this strain |
| `not_secreted` | Members present, none predicted to be secreted |
| `SP+` | At least one member is predicted to be secreted with a signal peptide |
| `SP-` | At least one member is predicted to be secreted without a signal peptide (unconventional) |
| combinations, **semicolon-separated** | e.g. `SP+;not_secreted`: the strain has both secreted and non-secreted members |

Orthogroup categories used in the paper are derived from these values across all strains where the orthogroup is present:

| Category | Rule | Count |
|---|---|---|
| Secretome only | at least one `SP+`/`SP-` and no `not_secreted` anywhere | 135 |
| Mixed | at least one `SP+`/`SP-` and at least one `not_secreted` | 2,945 |
| Non-secretome only | `not_secreted` only | 27,764 |

Secretome only + Mixed = 3,080 secretome-associated orthogroups (Table S1).

When parsing combined values, match whole tokens, e.g. `r"(?:^|;)SP\+(?:;|$)"`. A plain substring test cannot tell the tokens apart reliably.

The protein-level version (one row per protein) is `02_orthology/orthogroup_secretome_tagging_long.tsv`.

### `pfam_enrichment_secretome_vs_nonsecretome.tsv`

Per-strain two-sided Fisher's exact tests of each Pfam domain in predicted secreted proteins vs. non-secreted proteins, with Benjamini–Hochberg correction within each strain.

| Column | Description |
|---|---|
| `Pfam_acc`, `Pfam_name` | Pfam accession and name |
| `k_secretome`, `n_secretome`, `prop_secretome` | Secreted proteins carrying the domain, total secreted proteins, proportion |
| `k_nonsecretome`, `n_nonsecretome`, `prop_nonsecretome` | Same for non-secreted proteins |
| `odds_ratio`, `log2_OR` | Effect size; `log2_OR > 0` = enriched in the secretome |
| `p_value`, `q_value` | Fisher's exact *p*; BH-adjusted *q* |
| `min_count_filter` | Minimum-count threshold applied (`0` or `1`). The table stacks both runs, so **filter on one value before use** |
| `strain` | Strain key (*S. microadriaticum* is `Smic` here, not `Smic_jgi`) |

Significant = `q_value < 0.05`; enriched = significant and `log2_OR > 0`.

---

## Data stored elsewhere in the repository

| Content | Location |
|---|---|
| Secretome prediction outputs for every step, per strain (non-redundant proteomes, SignalP, DeepLoc, DeepTMHMM, TargetP, ASAFind, ER-motif and GPI filters) | `01_secretome_prediction/<strain>/01_cdhit/` … `08_gpi_filter/` |
| **Final predicted secretomes** (FASTA + CSV, SP+ and SP−) | `01_secretome_prediction/<strain>/09_final/<strain>_SP_pos\|SP_neg_secretome_final.faa` |
| Per-step protein counts | `01_secretome_prediction/<strain>/summary/<strain>_pipeline_stats_final.tsv` |
| OrthoFinder results (orthogroups, gene trees, species tree) | `02_orthology/` |
| dbCAN and MEROPS results for the secretomes | `03_functional_annotation/dbcan_out/`, `merops_out/` |
| Per-strain Pfam maps and enrichment inputs | `04_pfam_enrichment/` |
| Proteome Discoverer protein reports (exoproteome) | `05_exoproteome_overlap/20250411_2025G24_e{1,2,3}_Proteins.txt` |
| Secretion gain/loss reconstruction tables | `06_secretion_evolution/` |

The three exoproteome reports correspond to culture supernatants of `e1` = *S. microadriaticum*, `e2` = *S. natans*, and `e3` = *B. minutum*. These LC-MS/MS data include proteins released by cell lysis. They are used to describe detected proteins, not to validate the secretome predictions.

---

## External data (not in this repository)

| Data | Source |
|---|---|
| Input proteomes | accession in `config/strains.tsv` |
| LC-MS/MS raw files | <!-- TODO: jPOST / PRIDE accession --> |
| Archived release of this repository | Liu, Y. et al. (2026) “Symbiodiniaceae secretome analysis”. Zenodo. Available at: [https://doi.org/10.5281/zenodo.24190280](https://doi.org/10.5281/zenodo.24190280).  |

---

## Identifier formats

| Strain | Example ID | Note |
|---|---|---|
| *S. pilosum*, *S. natans*, *S. tridacnidorum*, *S. linucheae*, *S. necroappetens* | `CAE6908775.1` | Protein accessions from the source database |
| *S. natans* (exoproteome only) | `Snat_CCMP2548.gene10002.mRNA1` | Gene-model IDs in the MS search database; linked to the accessions above by MD5 hash of the protein sequence |
| *S. microadriaticum* | `jgi\|Symmic1\|10000_rna1384` | The last separator is an underscore; the MS report writes `jgi\|Symmic1\|31500\|rna4278` |
| *B. minutum* | `symbB.v1.2.000001.t1` | FASTA headers carry extra fields (`…\|scaffold\|size\|…`); the canonical ID is extracted with `symbB\.v1\.2\.\d{6}\.t\d+` |
