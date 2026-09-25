# Typhimurium: human versus animal source classification

Computational analyses of *Salmonella enterica* serovar Typhimurium genomes from human, poultry, bovine, and swine sources in North America. This project investigates whether functional gene presence/absence profiles distinguish human-derived from animal-derived isolates, and how population structure affects that discrimination.

The analysis combines wgMLST data processing, machine learning, phylogenetic sensitivity analyses, SHAP feature attribution, and functional and spatial interpretation. The primary analysis environment is **Google Colab**; **Snippy was run separately**.

## Dataset

Genomes and associated metadata were obtained from the [EnteroBase Salmonella database](https://enterobase.warwick.ac.uk/) for isolates collected in North America during **2020–2025**. Animal categories include isolates from animals and food products derived from the corresponding source.

Following metadata curation and SISTR serotyping/QC, the final dataset was balanced using random seed **42**:

| Source | Genomes | Classification label |
| --- | ---: | --- |
| Human | 699 | Human (`1`) |
| Poultry | 233 | Animal (`0`) |
| Bovine | 233 | Animal (`0`) |
| Swine | 233 | Animal (`0`) |
| **Total** | **1,398** | **699 human / 699 animal** |

The main dataset is `final_dataset_typhimurium_699human_699animals_seed42.tsv`. It contains isolate metadata and wgMLST allele calls. It is **not** the derived binary feature matrix used directly for model fitting.

For modelling, missing/invalid alleles and alleles flagged as truncated are coded as `0`; other valid alleles are coded as `1`. Invariant loci are removed, yielding **6,254 variable loci** in the reported analysis. Here, “functional presence” is an operational sequence-based definition, not an experimental measurement of gene activity.

## Code and data access

- **Repository:** [GiovanniMerici/Typhimurium_Human_vs_Animal](https://github.com/GiovanniMerici/Typhimurium_Human_vs_Animal)
- **Main notebook:** `pipeline.ipynb`
- **Supporting data and results:** [Salmonella_typhimurium_NA on Google Drive](https://drive.google.com/drive/folders/1LbWAsNppZAY4DYJSTDGuP98M1VtKn7Av?usp=sharing)

The supporting files are maintained in the Drive project folder. Access depends on its sharing permissions. Download the inputs needed for the sections you intend to run and preserve their folder names. **Cloning the GitHub repository does not download files linked from Google Drive.**

The directory map below describes the project data layout; it does not imply that every folder is bundled in the Git repository.

| Folder | Contents or role |
| --- | --- |
| `Human/`, `Poultry/`, `Bovine/`, `Swine/` | Source-specific metadata, wgMLST exports, and genome assemblies |
| `sistr_input_fasta/`, `sistr_results_per_genome/` | Prepared genome FASTAs and per-genome serotyping results |
| `sistr_tmp_per_genome/`, `sistr_results/` | SISTR working/intermediate directories |
| `dataset_statistics/`, `dataset_figures/` | Dataset summaries and figures |
| `grapetree/`, `pcoa/` | Allelic profiles, minimum-spanning-tree analysis, and ordination |
| `truncated_wgmlst/` | Allele truncation screening, corrected tables, and QC |
| `pca_binary/` | PCA of the binary profiles |
| `ml/` | Model inputs, performance tables, split assignments, SHAP outputs, and sensitivity analyses |
| `snippy/`, `itol/` | Separate phylogenetic workflow files and tree annotations |
| `reference_allele_1_multifasta/` | Reference allele sequences and translation outputs |
| `eggnog/`, `baktfold/`, `annotations_merged/` | Functional annotation outputs and merged annotation tables |
| `enrichment/` | Source- and sequence-type-specific frequency summaries |
| `bertopic/`, `bertopic_v2/`, `bertopic_st19/` | Annotation-text embeddings and topic analyses |
| `esm3/` | Exploratory protein embeddings and associated analyses |
| `spatial/`, `spatial_st19/` | Reference-genome mapping, spatial summaries, and figures |

## Running in Google Colab

### 1. Open the notebook

Open [Google Colab](https://colab.research.google.com/), select the GitHub option in the notebook-opening dialog, and enter the repository URL. Alternatively, download the notebook from GitHub and upload it to Colab. Save a working copy before changing paths or settings.

### 2. Make the data accessible

The notebook repeatedly uses this project directory:

```text
/content/drive/MyDrive/Salmonella_typhimurium_NA
```

Place the required data in an accessible folder at that location, or update **all relevant path definitions** in the notebook. Changing one `PROJECT_DIR` variable does not update the separate hard-coded paths elsewhere.

Mount your own Drive **before the first data-loading cell**:

```python
from google.colab import drive
drive.mount("/content/drive")
```

Mounting Drive does not automatically copy the project data. The notebook's existing mount cells occur later than the first data reads, so run a mount cell at the beginning of a fresh session.

### 3. Run the installation cells for the sections you need

Software installation commands are embedded throughout the notebook. A separate local Python setup is not required for the intended Colab workflow, but the notebook is **not a fully pinned environment specification**.

| Analysis | Setup present in the notebook |
| --- | --- |
| SISTR | `sistr_cmd`, `pandas`, `tqdm`; BLAST+ and MAFFT installed through `apt-get` |
| Compatibility setup | Explicit pins for `numpy==1.26.4` and `pandas==2.2.2` |
| Figures and ordination | Plotly, Kaleido, GrapeTree, and scikit-bio; a later cell pins `kaleido==0.2.1` |
| Sequence processing | Biopython and tqdm |
| Machine learning | XGBoost and SHAP |
| Topic modelling | BERTopic, sentence-transformers, umap-learn, HDBSCAN, scikit-learn, and matplotlib |
| Optional ESM3 exploration | `esm`, PyTorch, Biopython, pandas, PyArrow, tqdm, and huggingface_hub |
| Spatial analysis | Biopython and BLAST+ |

Some libraries are imported before their later installation cells, while others, including SciPy, rely on the runtime or transitive dependencies. Run the appropriate setup cells before entering a section. Package upgrades or reinstalls may require a runtime restart and reloading the section's variables.

The manuscript reports Python **3.12.13**, pandas **2.2.2**, SISTR **1.1.3**, Biopython **1.87**, XGBoost **3.3.0**, and SHAP **0.52.0**. These are reported study versions; most are not pinned by the notebook's installation commands, so a fresh installation may differ.

### 4. Select the compute environment

The model-comparison and full-dataset tuned XGBoost cells explicitly use `device="cuda"`. Select a Colab GPU runtime for these settings, or change them to `device="cpu"`. Several sensitivity-analysis cells already use CPU settings. Embedding sections also use a GPU when available.

Optional ESM3 cells use Hugging Face authentication. Use your own authorized account and provide credentials interactively or through Colab secrets; keep tokens out of notebook source and saved outputs.

### 5. Choose a starting point

| Goal | Required starting files |
| --- | --- |
| Rebuild the study cohort | Source-specific EnteroBase exports and genome assemblies in the four source folders |
| Rebuild the binary matrix | Final cohort TSV, GrapeTree allelic profiles, and allele FASTAs or the corrected truncation table |
| Rerun the classification analyses | `ml/input/wgmlst_binary_presence_absence_filtered_variable_loci_with_source_group.tsv` |
| Run sequence-type sensitivity analyses | The binary matrix above and the final cohort TSV |
| Run ST19 phylogeny-aware evaluation | Those tables plus `itol/snippy_gubbins_tree/typhimurium_1398_tree-1.treefile` |
| Explore functional/spatial results | The relevant saved annotation tables, locus lists, sequences, and reference files |

The truncation-screening and reference-allele-extraction cells use an additional directory **outside the main project folder**:

```text
/content/drive/MyDrive/salmonella_source_attribution_final_merged/wgmlst_allele_fastas
```

These allele FASTAs must be supplied separately, or both `FASTA_DIR` definitions must be changed to their actual location. The final cohort TSV alone is insufficient to recreate the truncation calls.

## Analysis workflow

1. **Dataset preparation:** merge EnteroBase exports, curate source metadata, apply SISTR serotyping/QC, and construct the balanced cohort.
2. **Population structure:** examine wgMLST relationships with GrapeTree and PCoA, and binary-profile variation with PCA.
3. **Feature construction:** identify truncated alleles, construct binary profiles, remove invariant loci, and attach source labels.
4. **Model comparison:** evaluate logistic regression, Random Forest, ExtraTrees, and XGBoost on the same 50 stratified 80/20 train/test splits.
5. **Tuned XGBoost and SHAP:** use nested tuning within the training partitions and calculate feature attribution on held-out genomes. Save performance, selected parameters, split membership, and SHAP summaries/matrices.
6. **Sensitivity analyses:** assess sequence-type baselines, within-ST19 performance, exclusion of ST36, size-matched subsets, and phylogeny-aware group validation.
7. **Interpretation:** combine supplied annotation and source-frequency tables with topic modelling and spatial analyses. ESM3 embedding analyses are additional exploratory sections.

The notebook is a section-oriented research record containing exploratory, diagnostic, and repair cells. Some cells depend on variables created earlier in a session. Run the relevant sections with their prerequisites loaded; the presence of installation cells alone does not guarantee an uninterrupted fresh-runtime “Run all” execution.

## Separate Snippy and phylogenetic workflow

Snippy was executed outside Google Colab. The manuscript describes a phylogenetic workflow using **Snippy 4.6.0**, **Gubbins 3.1.0**, **snp-sites**, and **IQ-TREE 3.0.1**, with *S. Typhimurium* LT2 assembly **GCF_000006945.2 (ASM694v2)** as the reference. Trees were visualized and annotated in **iTOL v6**.

The notebook consumes the resulting tree for downstream analyses; it does not include installation or execution commands for the complete phylogenetic workflow. Reusing the tree does not require rerunning Snippy. Rebuilding it requires a separate environment with the phylogenetic tools and original genome inputs. Tree-tip identifiers must match the assembly barcodes used by the notebook.

The manuscript also describes **eggNOG-mapper 2.1.15** and **Baktfold 0.1.0**. Their installation and annotation runs are not included in the notebook. Corresponding annotation outputs are inputs to the downstream analyses. Likewise, the notebook reads precomputed enrichment lists; it does not contain the complete Fisher-test and multiple-testing workflow described in the manuscript.

## Reported results and interpretation

The accompanying manuscript reports the following results (mean ± SD across repeated evaluations):

| Analysis | ROC-AUC |
| --- | ---: |
| Fixed-parameter XGBoost, full cohort | 0.949 ± 0.010 |
| Nested-tuned XGBoost, full cohort | 0.948 ± 0.010 |
| Fixed-parameter XGBoost, ST19 only | 0.951 ± 0.011 |
| ST19 phylogeny-aware evaluation, coarsest reported clustering | 0.657 ± 0.135 |

The lower performance when related isolates are kept together in validation indicates that population structure contributes to the predictive signal. Random-split performance should therefore be interpreted alongside the phylogeny-aware analyses. Associations with isolation source and SHAP contributions do not establish causal host adaptation or experimentally validated gene function.

## Reproducibility

Use the saved cohort and split-assignment files when comparing results with the study. Preserve isolate identifiers, feature columns, label encoding, and the input file versions. The notebook records random seeds, model settings, QC summaries, and intermediate outputs, but fixed seeds alone do not guarantee identical results across changed software versions or CPU/GPU environments.

Refer to the accompanying methods/results manuscript for the full study design and interpretation. Additional exploratory notebook sections should be distinguished from the analyses reported in the manuscript.
