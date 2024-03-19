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
<summary><b>Command Line Syntax</b></summary>

> Note the `-n` in the code above. This syntax (a dash followed by a letter) is what is used to pass computational parameters to a program that alter its function. Here, `-n` stands for name. Note that flags will often (but not always) have a shortened/abbreviated form with a single `-` character and a longer explicit form preceded by two `-` characters. For most packages, a list of command options can be found with either `-h` or `--help`. In bioinformatics, you'll spend a lot of time on help screens...

</details>

2. Now let's run trimmomatic: 
```sh
trimmomatic PE -threads 4 -phred33 1mAM1.fastq 1mAM2.fastq AM1P.fastq AM1U.fastq AM2P.fastq AM2U.fastq ILLUMINACLIP:adapters.fasta:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:150
```
## Assembly 
1. Now that we've removed our adaptors and poor quality base calls, let's go ahead and assemble our genome. First, begin by creating a new conda environment and install the genome assembly software "SPAdes" just as you did for Trimmomatic. Then, we'll run SPAdes with default paramers: 
```sh
mkdir assembly 
spades.py -1 AM1P.fastq -2 AM2P.fastq -o ./assembly --cov-cutoff auto  
```
<details>
<summary><b>Understanding the Assembly Process</b></summary>

>Many modern assemblers use a branch of mathematics called graph-theory. In graph theory, a graph is defined as a mathematical structure used to model the pairwise relationships between objects. In the context of genomic assembly, the objects whose relationships we desire to model are sequences of length $k$ called <b>k-mers</b>. These k-mers are generated from quality controlled, raw genomic sequences *in silico*. K-mers are then represented as vertices on a certain type of graph called a **De Bruijn Graph**. Two k-mers are then linked together by an edge if the suffix (ending) of the first k-mer matches the prefix (beginning) of the second. This process is repeated until each k-mer is visited once. This yields a sequential ordering of k-mers that *should* represent the correct sequential ordering of k-mer sequences within a genome. Often times, software will use dynamic programing algorithms to determine what value of $k$ is appropriate for a given set of reads, and may even adjust that value in real-time. 

</details>

2. The assembly process may take a while to finish running. Accordingly, we'll carry out the rest of our tutorial on a genome which has already been assembled ahead of time. Just as we performed QC on our raw reads, we will also want to do so on our assembled genome. Download the seqkit package in the same manner you downloaded SPAdes and Trimmomatic. Enter the command `seqkit` to bring up the help menue, and use the information provided there to determine how to run the `stats` functionality. 

<details>
<p style="margin-left: 25px;">
<summary><p style="margin-left;">Hint</p>
</summary>


```sh
seqkit stats NesPan3.fasta 
```

</details>
