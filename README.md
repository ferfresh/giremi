# giremi
GIREMI is a method that can identify RNA editing sites using one RNA-seq data set without requiring genome sequence data.  

### How GIREMI works?

GIREMI calculates the mutual information (MI) of the mismatch pairs identified in the RNA-seq reads to distinguish RNA editing sites and SNPs. It also trains a generalized linear model (GLM) to achieve enhanced predictive power, which makes use of sequence bias information and the difference between the mismatch ratio of the unknown single nucleotide variants (SNVs) and the estimated allelic ratio of the gene.

### Detailed information on GIREMI and citation

A paper describing GIREMI is published at Nature Methods.  


### System requirement

- Currently GIREMI supports Linux 64bit (Ubuntu, Red Hat, SUSE and others) system. 
- At least 8 GB of memory is required to process typical human data sets.

### Dependent libraries or software

- HTSlib : for accessing SAM/BAM files
Please make sure your dynamic library path in your configuration file includes the directory in which HTSlib is installed.
- samtools : for generating the faidx index of reference genome sequence
Please use “samtools faidx” to generate the index before running GIREMI.
- R : for general linear model training and prediction

### Installation

GIREMI was implemented using a combination of R, Perl and C codes.  

### Quickstart

Usage: giremi.pl [options] in1.bam [in2.bam [...]]

#### NOTE:   
-  The bam files should contain all final mapped reads in all chromosomes.
-  If multiple bam files are provided as input to GIREMI, they are handled as replicates that can be combined into one data set to generate one set of editing sites.  If it is desired that biological replicates be analyzed separately, each file should be run individually through GIREMI.

#### Input options:
+  -f, --fasta-ref    FILE   reference genome sequence file in fasta format (_NOTE: the faidx index file generated by samtools should be saved in the same directory as this fasta file) 
+  -l, --positions    FILE   the list of all filtered SNVs after removing likely sequencing errors or SNVs due to other artifacts (see our paper for details)
+  -o, --output       FILE   write output to FILE.res
+  -m, --min          INT    minimal number of total reads covering candidate editing sites  [default: 5]
+  -p, --paired-end   INT    1:paired-end RNA-Seq reads; 0:single-end [default: 1]
+  -s, --strand       INT    0:non-strand specific RNA-Seq; 1: strand-specific RNA-Seq and read 1 (first read for the paired-end reads) is sense to RNA; 2: strand-specific RNA-Seq and read 1 is anti-sense to RNA [default: 0]

Required format of the file containing the list of SNVs (-l option):

column 1 : The name of the chromosome or scaffold
column 2 : The starting position of the SNV in the chromosome or scaffold (0-based)
column 3 : The ending position of the SNV in the chromosome or scaffold (1-based)
column 4 : The name of the gene harboring this SNV; “Inte”: the SNV resides in the Intergenic region
column 5 : A flag, 1: the SNV belongs to dbSNP; 0: otherwise
column 6 : Strand (+ or -); “#” for “Inte” gene


Format of the output file:

NOTE: This output file includes a rich list of information about the SNVs. Not all sites in this file are predicted as RNA editing sites, see the ifRNAE field.

chr                     : Name of the chromosome or scaffold     
coordinate              : Position of the SNVs in the chromosome or scaffold (1-based)    
strand                  : Strand information
ifSNP                   : 1, If the SNV is included in dbSNP; 0: otherwise.
gene                    : Name of the gene harboring this SNV
reference_base          : The nucleotide of this SNV in the reference chromosome (+ strand)
upstream_1base          : The upstream neighboring nucleotide of this SNV in the reference chromosome (+ strand)
downstream_1base        : The downstream neighboring nucleotide of this SNV in the reference chromosome  (+ strand)
major_base              : The major nucleotide of the SNV in the RNA-seq data     
major_count             : Number of reads with the major nucleotide    
tot_count               : Total number of reads covering this SNV in the RNA-Seq data   
major_ratio             : The ratio of major nucleotide (major_count/tot_count)   
MI                      : The mutual information of this SNV if a value exists   
pvalue_mi               : P-value from the MI test if applicable    
estimated_allelic_ratio : Estimated allelic ratio of the gene harboring this SNV
ifNEG                   : 1: this SNV was a negative control in the training data  
RNAE_t                  : Type of RNA editing or RNA-DNA mismatches (A-to-G, etc)
A,C,G,T                 : Numbers of reads with specific nucleotides at this site
ifRNAE                  : 1: the SNV is predicted as an RNA editing site based on MI analysis; 
						  2: the SNV is predicted as an RNA editing site based on GLM 
						  0: the SNV is not predicted as an RNA editing site
