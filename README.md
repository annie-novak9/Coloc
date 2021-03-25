# GWAS eQTL Colocalization Pipeline
---
## Languages and Packages
---
**R**
```
library(dplyr)
library(glmnet)
library(reshape2)
library(methods)
library(doMC)
library(doRNG)
library(tidyr)
library(tibble)
```
**bash**
## Software ##
---
- [PLINK 1.9](https://www.cog-genomics.org/plink/)
- [COLOC 3.2-1](https://github.com/chr1swallace/coloc)

## Workflow ##
---
**Scripts 1a-c**<br />
These scripts use imputed transcript levels from the eQTL population to generate lists of significant SNPs that are within 1Mb of a predicted gene. The goal is to pull all significant cis-acting variants. This step can be executed by calling script 1c, which runs script 1b for every chromosome:<br />
	```
	chmod u+x 01c_run_chrom.sh<br />
	./01c_run_chrom.sh {path/to/SNP_annotation_file.txt} {path/to/gene_annotation_file.txt} {path/to/genotype_file.txt} {path/to/expression_file.txt} {list of population abbreviations}<br />
	#For any occurences of a population abbreviation or chromosome number in the file path, use "pop" and "chrom", respectively<br />
	```
Input Files:<br />
- SNP annotation file<br />
- Gene annotation file<br />
- Genotype file<br />
- Expression file<br />
Output files:<br />
- output/LD_matrix/{pop}/{pop}_chr_{chrom}_{gene}_1Mb_of_gene.txt<br />

**Script 2**<br />
This script takes in genotype dosage files in vcf file format and converts them to PLINK binary formats, .bed, .bim, and .fam. This step can be executed by calling script 2.<br />
        ```
        chmod u+x 02_make_bed.sh<br />
        ./02_make_bed.sh {pop} {path/to/directory/containing/population_folders_containing_dosage_files/}<br />
        ```
Input Files:<br />
- Genotype dosage files<br />
Output Files:<br />
- output/LD_matrix/{pop}/{pop}_chr{chrom}_dose.bed<br />
- output/LD_matrix/{pop}/{pop}_chr{chrom}_dose.bim<br />
- output/LD_matrix/{pop}/{pop}_chr{chrom}_dose.fam<br />

**Script 3**<br />
This script uses the significant SNP lists produced by script 1, along with the bfiles produced by script 2, to calculate LD matrices between every significant SNP around every predicted gene. This step can be executed by calling script 3.<br />
        ```
        chmod u+x 03_make_LD_matrix.sh<br />
        ./02_make_LD_matrix.sh {pop}<br />
        ```
Input Files:<br />
- output/LD_matrix/{pop}/{pop}_chr_{chrom}_{gene}_1Mb_of_gene.txt<br />
- output/LD_matrix/{pop}/{pop}_chr{chrom}_dose.bed<br />
- output/LD_matrix/{pop}/{pop}_chr{chrom}_dose.bim<br />
- output/LD_matrix/{pop}/{pop}_chr{chrom}_dose.fam<br />
Output Files:<br />
- output/LD_matrix/{pop}/{pop}_1Mb_coords_LDMatrix/{pop}_chr_{chrom}_{gene}_1Mb_LD.ld.gz<br />
- output/LD_matrix/{pop}/{pop}_1Mb_coords_LDMatrix/{pop}_chr_{chrom}_{gene}_1Mb_LD.snplist.gz<br />
