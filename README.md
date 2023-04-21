# Red Panda v. 0.1
### A tool for identifying variants in scRNA-seq
#### author: Adam Cornish <adam.cornish@unmc.edu> & Babu Guda <babu.guda@unmc.edu>
#### license: MIT

## Overview

Red Panda employs the unique information found in scRNA-seq to increase accuracy as compared to software designed for bulk sequencing. We utilize the fact that transcripts represented by scRNA-seq reads necessarily only originate from the chromosomes present in a single cell. Where applicable, this fact is used to decide what is and is not a heterozygous variant. For example, if 20% of the transcripts of a gene originate from the maternal chromosome and 80% originate from the paternal, then all the heterozygous variants of that gene in the expressed transcript will be represented at a reference to alternate allele ratio of either 1:4 or 4:1. In other words, all the heterozygous variants in that transcript are expected to be part of a bimodal distribution, which can be exploited to improve the accuracy of variant calling using scRNA-seq data. Such unique information could not be obtained from bulk sequencing, where each variant is independently called. As part of the process of identifying variants, Red Panda creates three different classes: homozygous-looking, bimodally-distributed heterozygous, and non-bimodally-distributed heterozygous.

To run Red Panda, follow these steps:
1. A standard RNA-seq pipeline needs to be run on the sequencing data to generate alignment files and quantification data. We recommend using the bcbio pipeline with STAR or hisat2 as the aligner.
2. Samtools mpileup needs to be run on the alignment files generated by hisat2/STAR to produce a vcf file.
3. Run Red Panda using the quant files from the bcbio run (final/$SAMPLE_NAME/salmon/quant.sf) and the vcf file generated by mpileup
4. Run GATK HaplotypeCaller with --intervals set to a bed file created from this VCF file: data/vcf/to_check/$SAMPLE_NAME.vcf

## Download and Install

    git clone --recursive git://github.com/adambioi/red_panda.git

## Usage
Red Panda requires a number of files to run properly and they are listed below.

    perl red_panda.pl -q sample_quant.sf -v sample_output.vcf -s sample_name -d base_output_directory -g (hg38|mm10) -A /path/to/genome/annotation.gtf -V

    -q sample_quant.sf          The expression file created by running sailfish or salmon on the alignment file for your sample.
    -v sample_input.vcf         The VCF file generated by samtools mpileup
    -s sample_name              The sample name to be used in the output files.
    -d base_output_directory    The base output directory. Additional directories will be created within this directory.
    -g (hg38|mm10)              The genome being used; currently only hg38 and mm10 are supported
    -A annotation.gtf           The Ensembl annotation file for the genome being used
    -V                          Verbose debug information

## Output
A base output directory is created using the supplied argument (-d; in this case, the base directory is "data") in which multiple subdirectories and files are generated:

    ├── data
    │   ├── bed: a bed file for each cell, containing the location of all of the variant calls that need to be further validated with GATK-HC
    │   ├── quant: a quant.sf file created by salmon/sailfish containing all of the expressed genes in each cell
    │   └── vcf
    │       ├── bad: the list of variants that have been ruled out by Red Panda
    │       ├── expressed: the list of all variant that exist in expressed genes as determined by salmon/sailfish
    │       ├── filtered: the list of variants generated by samtools mpileup that have had very basic filters added: DP >= 10 and not homozygous-looking
    │       ├── good: the list of heterozygous variants that are supported by the core Red Panda algorithm
    │       ├── homozygous: the list of homozygous-looking variants that have passed quality filters
    │       └── to_check: the list of heterozygous variants that need to be checked by GATK-HC

## Final Output
For a given cell, once the VCF file in the "to_check" folder has been assessed by GATK-HC, a final VCF file can be generated by combining the results from GATK-HC and those VCF files found in "data/good" and "data/homozygous".

## Software versions of 3rd party tools

### Samtools
As of right now, Red Panda works with samtools versions 0.1.19–1.8. However, if you are using version 1.9 of samtools, then you must switch to the bcftools mpileup command as it is the spiritual successor to samtools mpileup.

#### GATK HaplotypeCaller
Feel free to use the most up-to-date version of GATK HaplotypeCaller on the variants generated in the "data/vcf/to_check" directory.
