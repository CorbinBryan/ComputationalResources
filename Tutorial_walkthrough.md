## Downloading Files for Analysis 

## Quality Control 
Your first step in assembling a genome is to remove poor quality base calls and adaptor sequences from your raw DNA reads. To do so, we'll use a program called Trimmomatic, which can be easily installed using Conda. 

1. We'll first create a new conda environment called `trim` and download trimmomatic in that environment:
```sh
# create a new environment with name trim 
conda create -n trim 
# make trim the active environment (env to which pkgs installed)
conda activate trim 
# install trimmomatic from bioconda channel 
conda install -c bioconda trimmomatic 
```
<details>
<summary>Box 1: Command Line Syntax</summary>

> Note the `-n` in the code above. This syntax (a dash followed by a letter) is what is used to pass computational parameters to a program that alter its function. Here, `-n` stands for name. Note that flags will often (but not always) have a shortened/abbreviated form with a single `-` character and a longer explicit form preceded by two `-` characters. For most packages, a list of command options can be found with either `-h` or `--help`. In bioinformatics, you'll spend a lot of time on help screens...

</details>

2. Now let's run trimmomatic: 
```sh
trimmomatic PE -threads 4 -phred33 1mAM1.fastq 1mAM2.fastq AM1P.fastq AM1U.fastq AM2P.fastq AM2U.fastq ILLUMINACLIP:adapters.fasta:2:30:10
```