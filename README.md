# Typhimurium_Human_vs_Animal

Machine learning on wgMLST functional gene presence/absence to identify genes that distinguish *Salmonella* Typhimurium genomes recovered from human infections from those circulating in livestock reservoirs.


## Overview

1,398 *S.* Typhimurium genomes collected in North America from human, poultry, bovine and swine sources were retrieved from EnteroBase, confirmed as Typhimurium by in silico serotyping, and balanced between human origin and a pooled animal class (poultry, bovine and swine). wgMLST allelic profiles were converted into a binary matrix of functional gene presence, in which missing calls and alleles carrying premature stop codons are coded as absence. Supervised classification was then used as a gene-discovery framework, and the resulting candidate genes were characterised functionally and mapped to the reference genome.

This work follows the approach introduced in [Salmonella_Virulence](https://github.com/IZSLER-GenomicEpidemiology/Salmonella_Virulence), applied here within a single serovar.

## Contents

| Path | Description |
|---|---|
| `pipeline.ipynb` | Full analysis notebook: dataset construction, QC, population structure, feature matrix, machine learning, SHAP ranking, functional annotation and genomic localisation |


## Analysis steps

- **Dataset** — genome and metadata retrieval from EnteroBase; curation on source annotation; serovar confirmation with SISTR; balanced human/animal sampling
- **Population structure** — wgMLST minimum-spanning tree (GrapeTree), allelic-distance PCoA, and SNP-based maximum-likelihood phylogeny (Snippy, Gubbins, IQ-TREE)
- **Feature matrix** — screening of wgMLST allele sequences for internal stop codons; binary matrix of functional gene presence
- **Machine learning** — comparison of four classifiers over repeated stratified splits; tuned XGBoost with SHAP values computed on held-out genomes
- **Characterisation** — topic modelling of gene annotations (BERTopic), structure-informed protein annotation (Baktfold), BLASTN mapping and spatial clustering of enriched genes

## Requirements

Python 3.12 with pandas, NumPy, scikit-learn, scikit-bio, Biopython, XGBoost, SHAP, BERTopic, sentence-transformers, UMAP-learn, HDBSCAN, matplotlib and plotly. External tools: SISTR, GrapeTree, Snippy, Gubbins, snp-sites, IQ-TREE, BLAST+ and Baktfold. Exact versions are given in the manuscript Methods.

## Data availability

Genomes and wgMLST profiles are publicly available in EnteroBase (*Salmonella* database) and NCBI. 
