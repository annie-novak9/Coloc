# GWAS-eQTL Colocalization Pipeline
---
## Languages and Packages
---
**R**
```
library(dplyr)
library(reshape2)
library(glmnet)
library(methods)
library(doMC)
library(doRNG)
library(tidyr)
library(tibble)
```
**bash**

**Python**
```
import os
import os.path
import argparse
```
## Software ##
---
- [PLINK 1.9](https://www.cog-genomics.org/plink/)
- [COLOC 3.2-1](https://github.com/chr1swallace/coloc)

## Running the pipeline ##
---
### Notes ###
* The pipeline was written to only run one population at a time
* The user must cd into the 'Coloc' repo in order to properly run the pipeline
* If the phenotypes have dashes in the name, replace the dash with an underscore
* SNP annotation, genotype, and vcf files should be named with the convention 'chr#.' in relation to chromosome

### Flags for input files ###
```
Run GWAS-eQTL colocalization pipeline.

optional arguments:
  -h, --help            show this help message and exit
  --gwas GWAS           GWAS summary statistics directory path
  --eqtl EQTL           Exact eQTL txt file path
  --vcf VCF             Directory containing the vcf files from the eQTL
                        population
  --snp_annot SNP_ANNOT
                        SNP annotation files directory path
  --gene_annot GENE_ANNOT
                        Exact gene annotation file path
  --geno GENO           Genotype files directory path
  --frq FRQ             Exact frq file path
  --out OUT             Specify main output directory for all output files
  --pop1 POP1           Population used for vcf files
  --pop4 POP4           Population used for frq and eQTL files
  --pop_size POP_SIZE   Size of population
  --phenotypes [PHENOTYPES [PHENOTYPES ...]]
                        Phenotypes to test
  --chrs [CHRS [CHRS ...]]
                        Indicate what chromosomes to run
  -gene_id [GENE_ID [GENE_ID ...]]
                        Input specific genes ids to run colocalization on,
                        default is set to False

```
* User must place all GWAS summary statistic, vcf, SNP annotation, and genotype files into their respective directories and use those directory paths for the respective flags
* The default is running COLOC on all genes in the chromosomes chosen. However, the user can input a list of gene IDs under the optional --gene_id flag to only run COLOC on a specific subset of genes.

### Example Code ###
```
cd /homes/anovak9/Coloc
```
```
nohup python3 main_wrapper.py --gwas ~/COLOC_input_data/GWAS_SS --eqtl ~/COLOC_input_data/cis_eQTLs_AFA_WG_all_cis.txt.gz 
--vcf ~/COLOC_input_data/vcfs --snp_annot ~/COLOC_input_data/snp_annot --gene_annot ~/COLOC_input_data/annotation_all_aptamers_ENSG.txt 
--geno ~/COLOC_input_data/genotypes --frq ~/COLOC_input_data/AFA_prot_hg38.frq --out ~/coloc_output --pop1 ASW --pop4 AFA --pop_size 183 
--phenotypes BMI C_reactive HDL_cholesterol Total_cholesterol --chrs 1 2 3 &
```
* The wrapper should always be run as nohup (note beginning of command)
* In this example, the vcf files come from the population ASW and the frq and QTL files come from the population AFA and the pieline will only run chromsomes 1-3. In addition, phenotypes BMI, C_reactive, HDL_cholesterol, and Total_cholesterol will be run. 

## Workflow ##
---
### Scripts 1a-b ###
These scripts use imputed transcript levels from the eQTL population to generate lists of significant SNPs that are within 1Mb of a predicted gene. The goal is to pull all significant cis-acting variants. The main wrapper runs script 1b for every chromosome:

#### Input Files: ####
- Bim files from script 2
- SNP annotation file
- Gene annotation file
- Genotype file
- Populations
#### Output Files: ####
- out/LD_matrix/{pop}/{pop}\_chr\_{chrom}\_{gene}\_1Mb\_of\_gene.txt

### Script 2 ###
This script takes in genotype dosage files in vcf file format and converts them to PLINK binary formats, .bed, .bim, and .fam. This step can be executed by calling script 2.
```
chmod u+x 02_make_bed.sh 
./02_make_bed.sh {pop} {path/to/directory/containing/population_folders_containing_dosage_files/}
```
#### Input Files: ####
- Genotype dosage files
#### Output Files: ####
- out/LD_matrix/{pop}/{pop}\_chr{chrom}\_dose.bed
- out/LD_matrix/{pop}/{pop}\_chr{chrom}\_dose.bim
- out/LD_matrix/{pop}/{pop}\_chr{chrom}\_dose.fam

### Script 3 ###
This script uses the significant SNP lists produced by script 1, along with the bfiles produced by script 2, to calculate LD matrices between every significant SNP around every predicted gene. This step can be executed by calling script 3.
```
chmod u+x 03_make_LD_matrix.sh
./03_make_LD_matrix.sh {pop}
```
#### Input Files: ####
- out/LD_matrix/{pop}/{pop}\_chr\_{chrom}\_{gene}\_1Mb\_of\_gene.txt
- out/LD_matrix/{pop}/{pop}\_chr{chrom}\_dose.bed
- out/LD_matrix/{pop}/{pop}\_chr{chrom}\_dose.bim
- out/LD_matrix/{pop}/{pop}\_chr{chrom}\_dose.fam
#### Output Files: ####
- out/LD_matrix/{pop}/{pop}\_1Mb\_coords\_LDMatrix/{pop}\_chr\_{chrom}\_{gene}\_1Mb\_LD.ld.gz
- out/LD_matrix/{pop}/{pop}\_1Mb\_coords\_LDMatrix/{pop}\_chr\_{chrom}\_{gene}\_1Mb\_LD.snplist.gz

### Script 4 ###
This script takes the GWAS summary statistics, QTL, and frq files input by the user and formats them properly in preparation of COLOC analysis.
#### Input Files: ####
- Directory of GWAS summary statistic files
- Exact file path of QTL txt file
- Exact file path of frq file
#### Output Files: ####
- out/eQTL/{pop}/eQTL\_{pop}\_{phenotype}.txt.gz
- out/GWAS_TOPMED/{pop}/GWAS\_TOPMED\_{pop}\_{phenotype}.txt.gz

### Scripts 5a-b ###
These scripts take the input arguments from the user and run COLOC analysis for each population and phenotype specified by the user using the outputs from scripts 3 and 4. A summary table of the COLOC reults are written to an outfile.
#### Input Files: ####
- out/eQTL/{pop}/eQTL\_{pop}\_{phenotype}.txt.gz
- out/GWAS_TOPMED/{pop}/GWAS\_TOPMED\_{pop}\_{phenotype}.txt.gz
- out/LD_matrix/{pop}/{pop}\_1Mb\_coords\_LDMatrix/{pop}\_chr\_{chrom}\_{gene}\_1Mb\_LD.ld.gz
- out/LD_matrix/{pop}/{pop}\_1Mb\_coords\_LDMatrix/{pop}\_chr\_{chrom}\_{gene}\_1Mb\_LD.snplist.gz
#### Output Files: ####
- out/Coloc_output/{pop}\_{phenotype}.txt
