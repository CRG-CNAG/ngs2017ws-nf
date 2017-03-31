# NGS 2017 Post-Conference Workshop

## CalliNGS-NF
A Nextflow pipeline for basic Variant Calling Analysis with NGS RNA-Seq data

[![nextflow](https://img.shields.io/badge/nextflow-%E2%89%A50.24.0-brightgreen.svg)](http://nextflow.io)
[![CircleCI status](https://circleci.com/gh/CRG-CNAG/CalliNGS-NF.png?style=shield)](https://circleci.com/gh/CRG-CNAG/CalliNGS-NF/tree/master)

## Quickstart 

Install Nextflow by using the following command: 

    curl get.nextflow.io | bash 
    
Download the Docker image with this command (optional) : 

    docker pull cbcrg/callings-nf@sha256:b65a7d721b9dd2da07d6bdd7f868b04039860f14fa514add975c59e68614c310
    
Note: the Docker image contains all the required dependencies except GATK which 
cannot be included due to license restrictions. 

Download the `GenomeAnalysisTK.jar` package from [this link](https://software.broadinstitute.org/gatk/download/).    

Launch the pipeline execution with the following command: 

    nextflow run CRG-CNAG/CalliNGS-NF --gatk </path/to/GenomeAnalysisTK.jar>


## Pipeline Description

The RNA sequencing (RNA-seq) data, in additional to the expression information, can be used to obtain somatic variants present in the genes of the analysed organism. The CalliNGS-NF pipeline processes RNAseq data to obtain small variants(SNVs), single polymorphisms (SNPs) and small INDELs (insertions, deletions). The pipeline is an implementation of the GATK Best Practices for variant calling on RNAseq and includes all major steps of the analysis, [link](http://gatkforums.broadinstitute.org/gatk/discussion/3892/the-gatk-best-practices-for-variant-calling-on-rnaseq-in-full-detail). 

In addition to the GATK best practics, the pipeline includes steps to compare obtained SNVs with known variants and to calculate allele specific counts for the overlapped SNVs.

## Input files

The CalliNGS-NF pipeline needs as the input following files:
* RNAseq reads, `*.fastq`
* Genome assembly, `*.fa`
* Known variants, `*.vcf`
* Blacklisted regions of the genome, `*.bed`

The RNAseq read file names should match to the convention  `sampleID`[1|2]_{1,2}.fastq.gz 

where 
* **sampleID** is the identifier of the sample
* the first number **1,2,..** is the replicate ID
* the second number **1 or 2** is the read pair in the paired-end samples.


## Pipeline parameters

#### `--reads` 
   
* Specifies the location of the reads *fastq* file(s).
* Multiple files can be specified using the usual wildcards (*, ?), in this case make sure to surround the parameter string
  value by single quote characters (see the example below)
* By default it is set to the CalliNGS-NF's location: `$baseDir/data/reads/rep1_{1,2}.fq.gz`
* See above for naming convention of samples, replicates and pairs read files.

Example: 

    $ nextflow run main.nf --reads '/home/dataset/*_[1|2]_{1,2}.fq.gz'


#### `--genome`

* The location of the genome fasta file.
* It should end in `.fa`.
* By default it is set to the CalliNGS-NF's location: `$baseDir/data/genome.fa`.

Example:

    $ nextflow run main.nf --genome /home/user/my_genome/human.fa
    

#### `--variants`

* The location of the known variants VCF file.
* It should end in `.vcf` or `vcf.gz`.
* By default it is set to the CalliNGS-NF's location: `$baseDir/data/known_variants.vcf.gz`.

Example:

    $ nextflow run main.nf --variants /home/user/data/variants.vcf


#### `--blacklist`

* The location of the blacklisted genome regions in bed format.
* It should end in `.bed`.
* By default it is set to the CalliNGS-NF's location: `$baseDir/data/blacklist.bed`.

Example:

    $ nextflow run main.nf --blacklist /home/user/data/blacklisted_regions.bed


#### `--gatk` 
   
* Specifies the location of the GATK jar file.
* Download the `GenomeAnalysisTK.jar` package from [this link](https://software.broadinstitute.org/gatk/download/).
* By default is set to CalliNGS-NF's locations: `/usr/local/bin/GenomeAnalysisTK.jar`.

Example: 

    $ nextflow run main.nf --gatk /home/bin/GenomeAnalysisTK.jar


#### `--results` 
   
* Specifies the folder where the results will be stored for the user.  
* It does not matter if the folder does not exist.
* By default is set to CalliNGS-NF's folder: `results` 

Example: 

    $ nextflow run main.nf --results /home/user/my_results
    
    
## Pipeline results

For each sample with `sampleID` the pipeline creates a number of output files inside a current working folder.
Here is a brief description of output files:
* `sampleID/final.vcf`,  somatic SNVs called from the RNAseq data
* `sampleID/diff.sites_in_files`, comparison of the SNVs from RNAseq data with the set of known variants
* `sampleID/known_snps.vcf`, SNVs that are common between RNAseq calls and known variants
* `sampleID/ASE.tsv`, allele counts at a positions of SNVs (only for common SNVs)
* `sampleID/AF.histogram.pdf`, a histogram plot for allele frequency (only for common SNVs)


## Schematic Outline
![Image](../master/figures/workflow.png?raw=true)

## Requirements 

* Java 7/8
* [Docker](https://www.docker.com/) 1.10 (or higher) or [Singularity](http://singularity.lbl.gov) engine
* [GATK](https://software.broadinstitute.org/gatk/) 3.7 

Note: CalliNGS-NF can be used without a container engine by installing in your system all the 
required software components reported in the following section. See the included 
[Dockerfile](docker/Dockerfile) for the configuration details.
 

## Components 

CalliNGS-NF uses the following software components and tools: 

* Java 8 
* Picard 2.9.0
* Samtools 1.3.1
* Vcftools 0.1.14
* STAR 2.5.2b
* GATK 3.7
* R 3.1.1 
* Awk
* Perl
* Grep
