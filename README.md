# SILVI
A  pipeline to facilitate in T-cell epitope selection 

## Citation
The paper describing this pipeline is available [here](link)

## Installation

Clone the repository locally with the command:
```
git clone https://github.com/JoanaPissarra/SILVI2020.git
```

## Purpose

Merge and filter results of immunoinformatics tools to facilitate discovery of epitopes

## Workflow

![workflow](images/SILVI_workflow_2020.png)

### R package dependencies:

tidyverse
stringr
Peptides

### General execution process

The scripts Fire_classI.R and Fire_classII.R contain the commands to apply analysis steps to class I or class II epitope binding prediction data, respectively. Path to input files should be edited by the user in Fire_classI.R or Fire_classII.R. We recommend loading the script files in Rstudio (link to rstudio) and execute each step sequentially.

The script Fire.R contains the commands to integrate initial data and apply analysis steps. 
Path to input files should be edited by the user in Fire.R 
We recommand loading the script Fire.R in Rstudio (link to rstudio) and execute each step sequentially. 

The directory example contains a set of input files for both classes and the resulting output. 

### INPUT

FOR CLASS I:

First, users must provide results for proteins of interest generated by epitope binding programs.
Submit fasta sequences of proteins to the following online predictors
Currently, the script only accept results from 9-mer epitopes for class I alleles.

links to online predictor:

[For each allele-length combination, consensus method is used, which includes ANN, SMM, and CombLib. Moutaftsi, M et al. 2006. A consensus epitope prediction approach identifies the breadth of murine T(CD8+) -cell responses to vaccinia virus. Nat Biotechnol 24:817-819.](http://tools.iedb.org/mhci/)

[Jurtz, V et al. 2017. NetMHCpan-4.0: Improved Peptide–MHC Class I Interaction Predictions Integrating Eluted Ligand and Peptide Binding Affinity Data. J. Immunol. 199, 3360–3368](http://www.cbs.dtu.dk/services/NetMHCpan/)

[Rammensee, HG et al. 1999.SYFPEITHI: database for MHC ligands and peptide motifs. Immunogenetics 50: 213-219](http://www.syfpeithi.de)


Format your results files as csv 
* 4 columns: allele,peptides,seq_num,score
* comma separated
* Files name **MUST** be formated as: proteincode_predictorcode.csv 
* Move your files in a directory named data created in the working directory

FOR CLASS II: 

Similarly, user should obtained results from the online predictors.
Currently, the script only accept results from 15-mer  epitopes for Class II alleles.

links to online predictor:

[For each allele-length combination, consensus method is used, which includes NN, SMM, and CombLib: Wang, P et al. 2008. A systematic assessment of MHC class II peptide binding predictions and evaluation of a consensus approach. PLoS Comput Biol. 2008 Apr 4;4(4):e1000048. &  NN-align core and IC50 predictions: Wang, P et al. 2010. Peptide binding predictions for HLA DR, DP and DQ molecules. BMC Bioinformatics 11, 568](http://tools.iedb.org/mhcii/)

[Andreatta, M et al. 2015. Accurate pan-specific prediction of peptide-MHC class II binding affinity with improved binding core identification. Immunogenetics.67(11-12):641-50.](http://www.cbs.dtu.dk/services/NetMHCIIpan/)

Format your results files as csv 
* 6 columns: allele,seq_num,full_peptide,rank,core,ic50
* semi-colon separated
* Files name **MUST** be formated as: proteincode_predictorcode.csv 
* Move your files in a directory named data created in the working directory

### STEP A

To apply the first filtering step, SILVI imports and integrates all data into a single data frame class in R and directly compares all 9-mer peptides or cores among different sequences from the same protein (several seq_num per protein).
The script select 9-mer peptides or cores that are 100% identical (common_among_seq_nums filter). Simultaneously, the script selects only the 9-mer peptides or cores predicted by at least 2 predictors (common_among_predictors filter). 
All predicted HLA restriction are added to the exported table (new column: supertype) 

FOR CLASS I:
users open Fire_classI.R, introduce the pathway to input files and run the first code lines. To assign HLA restriction, the comparison is made per supertype (11 supertypes), which allows the comparison among predictors 50. 
In the file /code/map_supertype_alleles.csv, we find the correspondences between allele and supertypes (11), where users may add new alleles. 

HLA-class II - users open Fire_classII.R, introduce the pathway to input files and run the first code lines. To assign HLA restriction, SILVI compares per allele, so it is important to perform predictions with the same allele lists. 




### Intermediate Output

As an intermediary output, the script generates a .csv file for the initial epitope list (“1_common_I/II.csv”) with all the dataframe information so far (source protein, all predicted allele/supertype restrictions, peptide sequence, number of predictors, number of seq_nums, scores and raw data file). 
Also, a .txt file with all 9-mer peptides in FASTA format is generated, to be subsequently uploaded for online short-BLASTp analysis (“1_blast_me.fasta”).  
Users choose the host reference dataset (e.g. Homo sapiens taxid: 9606, RefSeq) and desired alignment parameters (e.g. default for short sequences). Once the alignment is complete, users download the short-BLASTp alignment result in .txt file format to be imported again in R for SILVI's step 2.


For  class I, the first step of the script output the following files:
* 1_common_I.csv
	Formatted prediction for class I supertype in a csv file.
* 1_blast_me_I.fasta
	Fasta file of peptide sequences to blast again host proteome.

For  class II, the first step of the script output the following files:
* 1_common_II.csv
	Formatted prediction for class II supertype in a csv file.
* 1_blast_me_II.fasta
	Fasta file of peptide sequences to blast again host proteome.


### Second and third selection steps
Users introduce the path and name of the short-BLASTp result file and run the code (STEP B). 
SILVI reads the first BLASTp alignment hit for each 9-mer peptide or core and counts the position-specific mismatches. Positive residues are considered a match, and when alignment gaps are introduced the succeeding positions are considered as mismatch.

Once the new dataframe is generated by the 2nd selection step, users run the last code lines (STEP C) and SILVI calculates the total number of mismatches in each peptide (“totalMM”), as well as supertype-specific anchor position mismatches for class I peptides (“anchorMM”). 
SILVI also runs the Peptides package on all 9-mer or 15-mer peptide sequences to add molecular weight (MW), isoelectric point (pI), hydrophobicity (Hy) and amino-acid composition information. 
Moreover, SILVI calculates the “promiscuity”, the total number of alleles/supertypes to which a given epitope is predicted to bind to, and duplicates the rows according to this information allowing selections based on HLA-restriction. 
Also, SILVI highlights the predicted IC50 by NetMHCpan algorithms (“scoreN”) to help the user select the top predicted binders.

For class I, the second step of the script output the following files:
* 2_common_blast_I.csv
* 3_blast_mismatches_I.csv

For class II, the second step of the script output the following files:
* 2_common_blast_II.csv
* 3_blast_mismatches_II.csv

### Final output

As a final output, SILVI generates a .csv file containing the initial peptide list from the first selection step, plus the short-BLASTp alignment results and all the other relevant information added in the last selection steps. 
The user is then free to analyse the list, complement with more data if needed, and prioritize the different criteria as desired.

For  class I, the last steps of the script output the following file:
* res_classI.csv

For  class II, the last steps of the script output the following file:
* res_classII.csv





# Getting started instructions

![getting started](images/read_me_getting_started.jpg)





