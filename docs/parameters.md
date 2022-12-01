---
title: Pipeline parameters
description: >-
    A complete list and description of PhyloNext parameters.
---

# Pipeline parameters

PhyloNext includes numerous configurable parameters.  
For convenience, they are grouped into several categories: 

- Input/output options
- Data subsetting:
    * Taxonomic scope
    * Spatial scope
- Spatial outliers removal
- Occurrence filtering and binning
- Diversity estimation
- Visualization
    * Interactive ([Leaflet](https://leafletjs.com/)-based)
    * Static (maps in pdf format)
- Phylogenetic tree-related parameters
- Generic options

## Input/output options

Define where the pipeline should find input data and save output data.

| Parameter   | Description                                                          | Type     | Default   |
| ----------- | -------------------------------------------------------------------- | -------- | --------- |
| `--input`   | Path to the directory with parquet files (GBIF occurrence dump in the Parquet format).<br>Could be stored locally or in the cloud (S3 or Azure Blob storage).                           | `directory` |           |
| `--outdir`  | The output directory where the results will be saved                 | `directory` | ./results |
| `--phytree` | Custom phylogenetic tree in Newick format (optional).<br>Tips should be labeled either with Latin binomials (e.g., "Homo_sapiens"), or with Open Tree IDs (e.g, "ott359899").<br>Please adjust the `--phylabels` parameter correspondingly.   | `file` |           |

## Taxonomic scope

Define which taxa should be analyzed.

| Parameter        | Description                                                                                      | Type      | Example            | Default |
| ---------------- | ------------------------------------------------------------------------------------------------ | --------- | ------------------ | ------- |
| `--phylum`       | Phylum to analyze [^1]                                                                           | `string`  | "Chordata"         |         |
| `--classis`      | Class to analyze [^1]                                                                            | `string`  | "Mammalia"         |         |
| `--order`        | Order to analyze [^1]                                                                            | `string`  | "Carnivora"        |         |
| `--family`       | Family to analyze [^1]                                                                           | `string`  | "Felidae,Canidae"  |         |
| `--genus`        | Genus to analyze [^1]                                                                            | `string`  | "Felis,Canis,Lynx" |         |
| `--specieskeys`  | Custom list of GBIF specieskeys (file with a single column)                                      | `file`    | "Carnivora"        |         |
| `--noextinct`    | File with extinct species specieskeys for their removal (file with a single column, with header) | `file`    | "Carnivora"        |         |
| `--excludehuman` | Exclude genus "Homo" from occurrence data                                                        | `boolean` | True               | True    |

[^1]: Multiple comma-separated values allowed (e.g., "Felidae,Canidae" for the Family rank).  

Unfortunately, `class` is a reserved keyword in Nextflow. Therefore, Latin `classis` is used as a parameter name.