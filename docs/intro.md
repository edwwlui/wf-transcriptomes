## Introduction

This workflow identifies RNA isoforms using either cDNA or direct RNA (dRNA) 
Oxford Nanopore reads.

### Preprocesing
cDNA reads are initially preprocessed by [pychopper](https://github.com/epi2me-labs/pychopper) 
for the identification of full-length reads, as well as trimming and orientation correction (This step is omitted for 
 direct RNA reads).


### Transcript assembly

#### Reference-aided transcript assembly approach
* Full length reads are mapped to a supplied reference genome using [minimap2](https://github.com/lh3/minimap2)
* Transcripts are assembled by [stringtie](http://ccb.jhu.edu/software/stringtie) 
in long read mode (with or without a guide reference annotation) to generate the GFF annotation.
* The annotation generated by the pipeline is compared to the reference annotation. 
using [gffcompare](http://ccb.jhu.edu/software/stringtie/gffcompare.shtml)

#### de novo-based transcript assembly (experimental!)
* Sequence clusters are generated using [isONclust2](https://github.com/nanoporetech/isONclust2)
  * If a reference genome is supplied, cluster quality metrics are determined by comparing    
  with clusters generated from a minimap2 alignment.
* A consensus sequence for each cluster is generated using [spoa](https://github.com/rvaser/spoa)
* Three rounds of polishing using racon and minimap2 to give a final polished CDS for each gene.
* Full-length reads are then mapped to these polished CDS.
* Transcripts are assembled by stringtie as for the reference-based approach.
* __Note__: This approach is currently not supported with direct RNA reads.

### Fusion gene detection
Fusion gene detection is performed using [JAFFA](https://github.com/Oshlack/JAFFA), with the JAFFAL extension for use 
with ONT long reads. 

### Differential expression analysis
* Differential expression is done using the transcripts output by the workflow.
* A non redundant transcriptome is found using the merge function in [stringtie](http://ccb.jhu.edu/software/stringtie).
* The reads are then aligned to the transcriptome using minimap2 in a splice-aware manner.
* [salmon](https://github.com/COMBINE-lab/salmon) is used for transcript quantification.
* R packages [edgeR](https://bioconductor.org/packages/release/bioc/html/edgeR.html) and [stageR](https://bioconductor.org/packages/release/bioc/html/stageR.html) are used for differential expression analysis.
* [DEXSeq](https://bioconductor.org/packages/release/bioc/html/DEXSeq.html) is then used for differential transcript usage analysis.

### Workflow inputs
- Directory containing cDNA/direct RNA reads. Or a directory containing subdirectories each with reads from different samples
  (in fastq/fastq.gz format)
- Reference genome in fasta format (required for reference-based assembly).
- Optional reference annotation in GFF2/3 format (required for differential expression analysis `--de_analysis`).
- For fusion detection, JAFFAL reference files (see Quickstart) 
