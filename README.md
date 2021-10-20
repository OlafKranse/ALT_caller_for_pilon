# ALT_caller_for_pilon
## Olaf Kranse and Prof. Andrew Bent
**Tested on: 4.19.0-17-amd64 #1 SMP Debian 4.19.194-3 (2021-07-18) x86_64 AND Ubuntu 20.04.3 LTS**

Currently does not work on windows 11, untested, but may run in WSL e.g. Ubuntu for windows https://ubuntu.com/tutorials/ubuntu-on-windows#1-overview

## Table of contents
	1) Rationale
	2) System requirements
	3) Installation guide
	4) Running the script
	5) Example running the script using screen

### 1) Rationale
This script was developed to aid in the phasing of haplotypes, and improved prediction of encoded protein variants, for DNA samples from inbred populations rather than single individuals. The script fills the ALT column of standard VCF files with the second-most-abundant nucleotide that was present at that position within the set of input DNA sequencing reads. In this way  that this second allele can be incorporated into haplotype genomes predicted for example by WhatsHap or Longphase software. 

Rationale: When genomic sites of single nucleotide variants are catalogued by variant calling software such as Pilon, the settings are understandably biased toward generating collapsed single-haplotype reference genomes, or for finding heterozygote alleles in DNA samples from diploid individuals.  Nucleotide abundance at any genomic position is expected to be close to 100% one nucleotide, or close to 50:50 for either of two nucleotides.  If the QP score (percentage of weighted evidence for each base A, C, G or T; sum = 100) for the most abundent nucleotide exceeds 75%, the software outputs only that one nucleotide into the REF column, leaves the ALT column blank, and sets genotype in the GT column as "0/0" indicating homozygosity for the most abundant nucleotide.  

In a more complex DNA sample (for example from an inbred population of obligate outbreeding plant pathogenic cyst nematodes), bona fide minor alleles can be present at frequencies below 25%.  ALT_caller_for_Pilon places the second-most abundant nucleotide in the ALT column so that subsequent haplotype phasing and protein predictions represent these minor-frequency alleles. 

ALT_caller_for_Pilon ignores (does not alter) VCF file lines in which a nucleotide is already present in the ALT column.  At present, ALT_caller_for_Pilon also ignores lines in which there is a tie for second (equal QP scores for second- and third-most abundant nucleotide).  

NOTE: ALT_caller_for_Pilon should only be used on filtered VCF files that have been purged of all data lines that do not pass other minimum-quality criteria (for example, requiring that lines lacking an ALT nucleotide must have a FILTER tag of PASS rather than LowCov or Del, and/or a valid read-depth exceeding 30 (DP>30), and/or QP score for the most abundant nucleotide <90% (so that alternative nucleotides are present in at least 10% of reads).  Other filters are possible and can be implemented using grep commands on the input vcf file before or after use of ALT_caller_for_Pilon. Users are cautioned to use sound assessment of the sequencing technologies used, the behavior and option-setting of the variant-calling software, and biological reasoning, as they balance false-positive and false-negative alternative allele inclusion.

### 2) System requirements

### 3) Installation guide
Most packages come with the default instalation of python3.



If you do not have pip:

	sudo apt update
	sudo apt install python3-pip

Once installed:

	pip3 install argparse


### 4) Running the script


**To run:**
1) Put both filter.py and replace.bash in the same folder as the VCF file you want to ALT call

2) run command: python3 filter.py -i "input file" -l "work dir"

  Required:
	
    [-i] "Input file" <- This is the VCF file given by pilon
		
  Optional:
	
    [-l] "Working dir"
		

3) After the scripts are done running, double check if the individual files created are your expected output and if so, run fuse.bash


### 5) Example

DIR start:
```bash
.
├── filter.py
├── fuse.bash
├── In_file.vcf
```
#### Running script 
	ok297:~/test$ python3 filter.py -i "In_file.vcf" -l "/home/ok297/test"
	Finding nucleotides...
	Checking file size...
	Splitting files, and submitting to nodes/screens...
**!Wait for all the nodes/screens to close!**

DIR after:
```bash
.
├── 100001-200000.txt <- This is the **second** 100k of the In_file.vcf
├── 100001-200000.txt_dir
│   ├── 100001-200000.txt
│   ├── out_file <- The 100k file, ALT called
│   ├── removing_missing
│   ├── replace.bash
│   ├── temp
│   └── test_file_for_check
├── 1-100000.txt <- This is the **first** 100k of the In_file.vcf
├── 1-100000.txt_dir
│   ├── 1-100000.txt
│   ├── out_file
│   ├── removing_missing
│   ├── replace.bash
│   ├── temp
│   └── test_file_for_check
├── filter.py
├── fuse.bash <- New bash script created by running filter.py, when you run it it makes the final file
├── In_file.vcf
├── in
├── out
├── setup.bash
├── temp
├── temp_head
```
#### Quality control
	ok297:~/test$ cd 1-100000.txt_dir
	ok297:~/test/1-100000.txt_dir$ wc -l 1-100000.txt <- Tis should be 100 000
	100000 1-100000.txt
	ok297:~/test/1-100000.txt_dir$ wc -l out_file <- This should be 100 001 (First line is empty (filtered out later), check by running head out_file)
	100001 out_file
	ok297:~/test/1-100000.txt_dir$ tail 1-100000.txt <- Not ALT called
	ok297:~/test/1-100000.txt_dir$ tail out_file <- same ends just alt called (if there are off by a few it's likely that there is an error in the temp file where the bases don't allign with the correct line number. Double check if your input file does not have any weird lines at the start)
**Check the rest of the files**

**If all looks okay**

	ok297:~/test/100001-200000.txt_dir$ cd ../
	ok297:~/test/$ bash fuse.bash

Final dir:
```bash
.
├── 100001-200000.txt 
├── 100001-200000.txt_dir
│   ├── 100001-200000.txt
│   ├── out_file 
│   ├── removing_missing
│   ├── replace.bash
│   ├── temp
│   └── test_file_for_check
├── 1-100000.txt
├── 1-100000.txt_dir
│   ├── 1-100000.txt
│   ├── out_file
│   ├── removing_missing
│   ├── replace.bash
│   ├── temp
│   └── test_file_for_check
├── filter.py
├── fuse.bash <- New bash script created by running filter.py, when you run it it makes the final file
├── In_file.vcf
├── in
├── out
├── setup.bash
├── temp
├── temp_head
├── pre_final
├── final <- the output file, same as your In_file.vcf, but ALT called
```
	


