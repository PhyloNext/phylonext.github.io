---
title: Usage instructions
description: >-
    An overview of PhyloNext parameters you might need to adjust to meet your needs.
---

## Input data

!!! info "Input data"
    For mode details, see the [input data](inputdata.md) description section in the documentation.

## Run test example

Test run the pipeline using test data and Docker engine:
``` bash
nextflow run vmikk/PhyloNext -r main -profile test,docker
```

## Running the pipeline

The typical command for running the pipeline is as follows:  

``` bash
nextflow run vmikk/PhyloNext -r main \
  --input "/mnt/GBIF/Parquet/2022-01-01/occurrence.parquet/" \
  --classis "Mammalia" --family  "Felidae,Canidae" \
  --country "DE,PL,CZ"  \
  --minyear 2000  \
  --dbscan true  \
  --iterations 100  \
  --outdir "$PWD" \
  -resume \
  -profile docker
```
This will launch the pipeline with the `docker` configuration profile (see below for more information about profiles).

!!! info "Parameters"
    All parameters are listed in the [`Parameters`](parameters.md) section of the documentation.  

Note that the pipeline will create the following files in your working directory:  

| Directory or file       | Description                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| `00.filtered1.parquet`  | Parquet subset of the data                                                                  |
| `01.filtered2`          | Data with spatial outliers removed, individually for each species in the dataset            |
| `01.NumRecords`         | Counts of the number of records per grid cell                                               |
| `02.Biodiverse_input`   | Table with species occurrences and phylogenetic tree, as well as input files for Biodiverse |
| `02.Biodiverse_results` | Biodiverse output                                                                           |
| `03.Plots`              | Data visualizations                                                                         |
| `work`                  | Directory containing the Nextflow working files                                             |
| `results`               | Pipeline execution reports and dataset DOIs                                                 |
| `.nextflow_log`         | Log file from Nextflow                                                                      |

For more details about output files, see the [Outputs](outputs.md) section of documentation.

## Workflow caching and checkpointing

All intermediate results of the pipeline run are stored in the `work` and `.nextflow` directories. 
To restart a pipeline following an error from the most recent successfully executed process, use the '-resume' flag. 
This way, Nextflow will use cached results from any pipeline steps where the inputs are the same, continuing from where it strop previously. 
This option could also be helpful if you wish to change some of the parameters of downstream processes and will allow reusing results of upstream steps.  

Without `-resume`, Nextflow will start the workflow from scratch and overwrite all previous results.  

## Built-in data

PhyloNext pipeline contains several files for removing spatial outliers or data subsetting.  
By default, built-in data is stored in the folder `${HOME}/.nextflow/assets/vmikk/PhyloNext/pipeline_data/`.

| File                                  | Description                                             | Citation                                                                                      |
| ------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `Land_Buffered_025_dgr.RData`         | Land polygon for removal of non-terrestrial occurrences | [Natural Earth](https://www.naturalearthdata.com/)                                            |
| `CC_CountryCentroids_buf_1000m.RData` | Polygons with country and province centroids            | [Zizka et al., 2019](https://besjournals.onlinelibrary.wiley.com/doi/10.1111/2041-210X.13152) |
| `CC_Capitals_buf_10000m.RData`        | Polygons with country capitals                          | [Zizka et al., 2019](https://besjournals.onlinelibrary.wiley.com/doi/10.1111/2041-210X.13152) |
| `CC_Institutions_buf_100m.RData`      | Polygons with biological institutions and museums       | [Zizka et al., 2019](https://besjournals.onlinelibrary.wiley.com/doi/10.1111/2041-210X.13152) |
| `CC_Urban.RData`                      | Polygons with urban areas                               | [Natural Earth](https://www.naturalearthdata.com/)                                            |
| `WGSRPD.RData`                        | Polygons of World Geographical Regions                  | [WGSRPD](https://www.tdwg.org/standards/wgsrpd/)                                              |
| `WorldMap_NaturalEarth_Medium.RData`  | World map                                               | [Natural Earth](https://www.naturalearthdata.com/)                                            |

To use the built-in data, provide a full path to the data.  
E.g., to remove records from urban areas, add the following parameter to your command:  
`--rmurban ${HOME}/.nextflow/assets/vmikk/PhyloNext/pipeline_data/CC_Urban.RData`.  
Alternatively, you may use custom files prepared in the same format.

To remove common spatial errors from the data, PhyloNext uses databases 
from the [`CoordinateCleaner`](https://ropensci.github.io/CoordinateCleaner/) R package by Alexander Zizka. 
The following buffers were added to the datasets because coordinates in a database could be slightly inaccurate due to errors: 

- Country and province centroids - 1 km buffer  
- Capital coordinates - 10 km buffer  
- Coordinates of biodiversity institutions - 100 m buffer  


For the removal of non-terrestrial samples, PhyloNext uses a shapefile from [Natural Earth](https://www.naturalearthdata.com/),
with a 0.25-degree buffer (corresponding to ~27 km distance, which is approximately half of the H3 grid cell at resolution 4).  

Please cite any datasets if you have used them in your analysis.  


## Workflow processes

Workflow managers such as Nextflow are designed to help users run complex pipelines of processes, 
potentially involving multiple steps and dependencies between them, efficiently and reproducibly. 
One of the key features of such tools is the ability to run processes in parallel, 
which can reduce the overall time taken to complete a workflow by distributing the workload across multiple computational resources. 
To run the pipeline effectively and maximize resource utilization, Nextflow will:  

- Effectively and automatically allocate CPUs on the available computational resources (local, a cluster, or a cloud environment);
- Use a scheduling algorithm to determine the order in which processes should be run based on the dependencies between them and the availability of resources.

During the pipeline execution, you will see a progress indicator in the output in your terminal:

```
[ad/02689a] process > occ_filter            [100%] 1 of 1 ✔
[b7/48eb93] process > record_count          [100%] 1 of 1 ✔
[cc/af139c] process > outl_low              [100%] 1 of 1 ✔
[88/197e96] process > outl_high (5219243)   [100%] 6 of 6 ✔
[85/8425c9] process > merge_occ             [100%] 1 of 1 ✔
[b6/bcf57e] process > prep_biodiv           [100%] 1 of 1 ✔
[aa/bbb119] process > phylodiv (1)          [100%] 1 of 1 ✔
[18/d367ac] process > rand_filelist         [100%] 1 of 1 ✔
[6b/b1d08b] process > aggregate_rnds_biodiv [100%] 1 of 1 ✔
[c6/ecf435] process > div_to_csv            [100%] 1 of 1 ✔
[ae/7de318] process > plot_pd               [100%] 1 of 1 ✔
[ba/141853] process > plot_leaflet          [100%] 1 of 1 ✔
[6a/ca28a8] process > derived_datasets      [100%] 1 of 1 ✔
```

Processes are run in parallel and start as soon as their input data becomes available. 
Pipeline scripts are executed by Nextflow in a separate working directory. 
Each working directory contains input files (symbolic links in case of running the pipeline locally), as well as results and log files.

!!! info "Log files"
    In case of pipeline failure, it could be useful to inspect logs for a failed task.  
    E.g., working directory for the `occ_filter` is located in `./work/ad/02689a...`  and contains a log file `.command.log`.  

Processes description:  

| Process name            | Description                                                                                              | CPUs per task ^1^ |
| ----------------------- | -------------------------------------------------------------------------------------------------------- | ----------------- |
| `occ_filter`            | Occurrence filtering of parquet file                                                                     | 4                 |
| `record_count`          | Counting the total number of records per H3 cell                                                         | 4                 |
| `outl_high`             | Per-species outlier removal for species with high occurrence (optionally, with DBSCAN), spatial binning  | 1                 |
| `outl_low`              | Spatial outlier removal for species with high occurrence (DBSCAN disabled), spatial binning              | 4                 |
| `prep_ott_ids`          | Preparation of Open Tree IDs for a phylogenetic tree (optional)                                          | 4                 |
| `get_ott_tree`          | Automatic fetching of a phylogenetic tree from Open Tree of Life (optional)                              | 1                 |
| `merge_occ`             | Merging filtered species occurrences, species name matching, phylogenetic tree pruning                   | 4                 |
| `prep_biodiv`           | Creation of Biodiverse input files and estimation of the observed diversity                              | 1                 |
| `phylodiv`              | Randomization of species communities and estimation of standardized effect sizes of biodiversity metrics | 1                 |
| `rand_filelist`         | Preparation of randomization results                                                                     | 1                 |
| `aggregate_rnds_biodiv` | Randomization results pooling                                                                            | 1                 |
| `div_to_csv`            | Export of Biodiverse results into CSV format                                                             | 1                 |
| `plot_pd`               | Plotting diversity indices (non-interactive maps in PDF format)                                          | 1                 |
| `plot_leaflet`          | Plotting diversity indices (interactive maps - Leaflet-based choropleth)                                 | 1                 |
| `derived_datasets`      | Preparation of a list of DOIs for the GBIF datasets involved in the analysis (optional)                  | 4                 |

^1^:
    The default number of CPUs per task is configured in the `nextflow.config`.

Processes are executed by Nextflow independently and are isolated from each other, 
which allows running the pipeline on different nodes at HPC or a cloud platform.
Most workflow processes support multi-threading (e.g., species occurrence filtering). 
In some cases, the task will be divided into chunks and run in parallel 
(e.g., estimation of standardized effect sizes - 10 processes with 100 randomization iterations each). 
It is possible to customize the number of CPUs required by the process task (see `Configuration` below). 
The total number of CPUs allowed to use by PhyloNext can be adjusted with `-qs` parameter (by default, all available CPUs will be used).

## Getting help

To show a help message, run:
``` bash
nextflow run vmikk/PhyloNext -r main --help
```

## Configuration and profiles

Nextflow has a very flexible pipeline configuration system that supports multiple options 
such as parameter files, configuration files, and profiles.  

### Parameter file

As the number of parameters can be quite large, it is possible to pass the pipeline parameters via YAML or JSON file, e.g.:
``` bash
nextflow run vmikk/PhyloNext -r main -resume -params-file Mammals.yaml
```

The YAML file could contain the following:
``` yaml
input      : "/mnt/GBIF/Parquet/2022-01-01/occurrence.parquet/"
classis    : "Mammalia"
family     : "Felidae,Canidae"
country    : "DE,PL,CZ"
minyear    : 2000
dbscan     : true
phytree    : "/path/to/the/phylogenetic/tree.nwk"
iterations : 100
```

### Configuration file

While the default requirements set within the pipeline will hopefully work for most people and with most input data, 
you may want to customize the compute resources that the pipeline requests. 
Each step in the pipeline has a default set of requirements for the number of CPUs 
(and optionally, RAM and execution time - which could be important for executing PhyloNext on the HPC cluster).  

To adjust the resource usage (e.g., the number of CPUs required by the `occ_filter` process), 
you may create a custom configuration file (e.g., `my.config`) of the form:  

``` java
process {

    // Occurrence filtering, stage I
    withName:occ_filter{
        cpus = 10
    }
}
```

Then, use the `-c` option to choose the created configuration file while running the pipeline. E.g.,  
``` bash
nextflow run vmikk/PhyloNext ... -c my.config
```

!!! info "Configs"
    For more information, see the Nextflow documentation - [Configuration](https://www.nextflow.io/docs/latest/config.html#configuration-file).

!!! tip "Multiple configs"
    When a pipeline is launched, Nextflow looks for configuration files (and profiles) in multiple locations. 
    The configs will be merged, and the settings in the config with higher priority will override the settings in the other configs. 
    If you want to ignore any default configuration files and use only the custom one, use the `-C your.config` option (please note that the `C` letter is capitalized). 
    For more details, see the [Configutation](https://www.nextflow.io/docs/latest/config.html#configuration) section in the Nextflow docs.

### Profiles

Nextflow profiles can be used as configuration presets for different compute environments.  

We highly recommend using Docker or Singularity containers for full pipeline reproducibility. 
For this, several generic profiles are bundled with the pipeline, instructing PhyloNext to use software that is packaged in the containers.  
Available profiles include:

| Profile name   | Description                                                                                                 |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| `docker`       | A generic configuration profile to be used with [Docker](https://docker.com/)                               |
| `singularity`  | A generic configuration profile to be used with [Singularity](https://sylabs.io/docs/)                      |
| `test` ^1^     | A profile with a complete configuration for automated testing (built-in phylogenetic tree)                  |
| `test_ott` ^1^ | A profile with a complete configuration for automated testing (phylogenetic tree will be fetched from OToL) |

^1^:
    Test profiles include links to test data and do not require any other parameters to be specified.

If `-profile` is not specified, the pipeline will run locally and expect all software to be installed and available on the `PATH`. 
This is _not_ recommended.  

It is worth noting that multiple profiles can be loaded, for example: `-profile test,docker`. 
Please keep in mind that the order of arguments is important! 
Profiles are loaded in sequential order so that the later profile will overwrite earlier profiles.  

!!! info "Cluster and cloud executors"
    To create a profile for running PhyloNext on HPC (e.g., using SLURM) or cloud (e.g., Azure Batch or AWS Batch), 
    please address Nextflow documentation - [Executor section](https://www.nextflow.io/docs/latest/executor.html)

## The other helpful commands

To make sure that you're running the latest version of the pipeline, 
you may download the latest version and update the pipeline using:  
``` bash
nextflow pull vmikk/PhyloNext
```
By default, the cached version of the pipeline is stored in the `~/.nextflow/assets/vmikk/PhyloNext` directory.

Run the latest development version of the pipeline (from the `main` branch of the code repository):
``` bash
nextflow run vmikk/PhyloNext -r main ...
```

When executing PhyloNext on your data, it is a good idea to specify a version number for the pipeline. 
This ensures that specific code revision and software dependencies are used. 
Even if the codebase has changed, if you continue using the same tag, you will run the same pipeline version.  
To find the latest version number, go to the [PhyloNext releases page](https://github.com/vmikk/PhyloNext/releases). 
Then, when launching the pipeline, specify the version using `-r` flag (e.g., `-r v.1.0.0`).
This version number will be recorded in pipeline reports so you can later go back and know what version you used.  

Run the tagged version (e.g., v.1.3.0) of the pipeline:
``` bash
nextflow run vmikk/PhyloNext -r v.1.3.0 ...
```

Print the pipeline and system runtime information:
``` bash
nextflow info
nextflow info vmikk/PhyloNext
```

Delete the local copy of the pipeline:
``` bash
nextflow drop vmikk/PhyloNext
```

## Docker and Singularity containers

Ready-to-use containers with all software dependencies required for PhyloNext 
are hosted on [Docker Hub](https://hub.docker.com/r/vmikk/biodiverse) and 
[Singularity Library](https://cloud.sylabs.io/library/vmiks/gbif/biodiverse).  

If you need to use a different version of a tool with the pipeline, 
you may re-build the default containers and override the location of a container in a custom configuration file. 
PhyloNext currently relies on three containers and refers to them using process labels.  

| Process Label          | Default Docker container | Docker file                                                                             | Default Singularity container           | Singularity definition file                                                                  |
| ---------------------- | ------------------------ | --------------------------------------------------------------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------- |
| `container_r`          | `vmikk/rarrow:1.3.0`     | [Link](https://github.com/vmikk/PhyloNext/blob/main/containerfiles/R_arrow.dockerfile)  | `library://vmiks/gbif/rarrow:1-3-0`     | [Link](https://github.com/vmikk/PhyloNext/blob/main/containerfiles/R_arrow_Singularity.def)  |
| `container_biodiverse` | `vmikk/biodiverse:1.3.0` | [Link](https://github.com/vmikk/biodiverse-docker/blob/main/Dockerfile_NoPerlbrew)      | `library://vmiks/gbif/biodiverse:1-3-0` | [Link](https://github.com/vmikk/biodiverse-docker/blob/main/SingularityDef_NoPerlbrew.def)   |
| `container_ott`        | `vmikk/opentree:0.0.2`   | [Link](https://github.com/vmikk/PhyloNext/blob/main/containerfiles/OpenTree.dockerfile) | `library://vmiks/gbif/opentree:0-0-2`   | [Link](https://github.com/vmikk/PhyloNext/blob/main/containerfiles/OpenTree_Singularity.def) |

The configuration file has the following format and must be adjusted if you would like to use custom containers:  

``` java
process {

    withLabel: 'container_r' {
        container = 'vmikk/rarrow:1.3.0'
    }

    withLabel: 'container_biodiverse' {
        container = 'vmikk/biodiverse:1.3.0'
    }

    withLabel: 'container_ott' {
        container = 'vmikk/opentree:0.0.2'
    }

}
```
Containers can either be pulled from the web (e.g., use `youraccount/containername` to get it from Docker Hub), 
or you may use local files (specify tag name for Docker, or a full path to the Singularity file).  

### Docker image

To build the [Docker](https://www.docker.com/) image with Biodiverse (`container_biodiverse`), run:
``` bash
git clone https://github.com/vmikk/biodiverse-docker
cd biodiverse-docker
docker build --tag biodiverse --file Dockerfile_NoPerlbrew . 
```

### Singularity image

To build the [Singularity](https://sylabs.io/singularity/) image with Biodiverse (`container_biodiverse`), run:
``` bash
git clone https://github.com/vmikk/biodiverse-docker
cd biodiverse-docker
sudo singularity build Biodiverse.sif SingularityDef_NoPerlbrew.def
```


## System requirements

When running the pipeline, it's important to consider the system requirements and time elapsed to complete the analysis.  

For example, for the global analysis of Mammal diversity at H3 resolution = 3 
(4710 species and 9814 grid cells in total),
and using 5000 spatially-uncontrained randomizations, with DBSCAN removal of outliers, 
the total CPU hours required was 113.6 hours (using 50 CPUs, the wall time was 4 hours). 
The peak memory usage during the analysis was 62GB for a DBSCAN filtering of occurrences of a very abundant species. 
This analysis is computationally intensive and requires a powerful computer with at least 64GB of RAM.

In contrast, the same analysis with a basis of record restricted to `PRESERVED_SPECIMEN` 
(4333 species and 7182 grid cells) 
required 30.4 CPU hours and a wall time of 1 hour and 13 minutes. 
The maximum memory usage was 9GB for an interactive map generation.

For a smaller-scale analysis, usually a less powerful computer with at least 16GB of RAM should be sufficient.


## Running on HPC

If you would like to run PhyloNext on HPC cluster, Nextflow can handle job submissions and supervise the running jobs. 
In this case the Nextflow process must run until the pipeline is finished. 
For this purpose you may either use `tmux` or `screen` to create a detached terminal session which you can log back into at a later time. 
Or you may submit Nextflow process using your job scheduler (e.g., SLURM), from where Nextlow will start submitting the other jobs. 
You can also use the `-bg` flag to launch Nextflow in the background to avoid blocking your current session.  

Depending on the number of tasks, in some cases, the Nextflow Java virtual machines can require a large amount of memory. 
To specify the RAM limits, you may create the following environmental variable before running Nextflow:  

``` bash
NXF_OPTS='-Xms1g -Xmx4g'
```

## What's next?

- Explore interactive maps (see [Output](outputs.md) file description)  
- Manually configure static map using the output in GeoPackage format (see [Postprocessing](post.md))  
- Explore the results using Biodiverse program (see [Postprocessing](post.md))  
- Prepare citable derived dataset with unique DOI (see [Postprocessing](post.md))  
