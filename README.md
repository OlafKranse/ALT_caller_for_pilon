# ALT_caller_for_pilon
**Tested on: 4.19.0-17-amd64 #1 SMP Debian 4.19.194-3 (2021-07-18) x86_64 AND Ubuntu 20.04.3 LTS**
Currently does not work on windows 11 unless run in WSL e.g. Ubuntu for windows https://ubuntu.com/tutorials/ubuntu-on-windows#1-overview

## Table of contents
	1) Running the script
	2) Example running the script using screen


### 1) Running the script


**To run:**
1) Put both filter.py and replace.bash in the same folder as the VCF file you want to ALT call

2) run command: python3 filter.py -i "input file" -l "work dir"

  Required:
	
    [-i] "Input file" <- This is the VCF file given by pilon
		
  Optional:
	
    [-l] "Working dir"
		

3) After the scripts are done running, double check if the individual files created are your expected output and if so, run fuse.bash


### 2) Example

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
	ok297:~/test/1-100000.txt_dir$ tail 1-100000.txt <- same ends just alt called
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
	


