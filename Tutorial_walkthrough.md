## Downloading Files for Analysis 
* For this tutorial, you will assemble and analyze a genome produced with Illumina 150 bp paired-end sequencing from a preserved specimen of *Amanita muscaria*, a mushroom-forming basidiomycete fungus (Amanitaceae, Agaricales). 
* Please do not share the provided tutorial genome. 
* Access to the raw sequence files are available via a Google-Drive link which I will provide either during a tutorial workshop or by request. 
* When you have been granted access and have downloaded the tutorial dataset, clone this repository onto your Desktop:
```sh
git clone https://www.github.com/CorbinBryan/ComputationalResources.git 
```
* Use the `mv` command to move your compressed tar-ball (containing the raw sequencing reads) into this directory. 
```sh
# mv moves a file from a source to a destination
# syntax: mv /path/to/source/file.txt /path/to/destination
# if a file is specified as destination rather than a directory
#       the command can be used to rename files 
mv /path/to/tarball/AM.tar.gz /path/to/ComputationalResources
```
* Decompress the provided tar-ball: 
```sh
tar -xzvf AM.tar.gz 
```
* Navigate into the resulting subdirectory using the `cd` command. In Unix like languages, `./` tells the interpreter that a provided directory is a subdirectory of the current working directory. To view the location of the current working directory, you can use the command `pwd`, which outputs the path to the working directory to standard out. To navigate to the parent directory of a given directory (i.e., the folder that holds a folder), you can use `../`. While in that directory, examine the structure of your raw sequencing files before returning to the parent directory.
```sh
cd ./1AM 
# the head command will view the first few lines of a given text file 
head 1mAM1.fastq 

cd .. 
```
* How does this FASTQ file compare to a typical FASTA file? What information is lacking in a FASTA file but provided in a FASTQ file? 
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
trimmomatic PE\     # Run trimmomatic in paired end mode
 -threads 4\        # Run with 4 threads
 -phred33\          # Quality scores encoded with PHRED33 
 1mAM1.fastq\       # Input file (forward)
 1mAM2.fastq\       # Input file (reverse)
 AM1P.fastq AM1U.fastq AM2P.fastq AM2U.fastq\ # output files
 ILLUMINACLIP:adapters.fasta:2:30:10\ # Path to adaptor seqs 
 LEADING:3\ # remove leading bases with qual below 3
 TRAILING:3\ # remove trailing bases with qual below 3 
 SLIDINGWINDOW:4:15\    # Sliding window length of 4 bp with qual threshold of 15 (below 15 = removed) 
 MINLEN:150     # Minimum read length 
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
<summary>Hint</summary>


```sh
seqkit stats NesPan3.fasta 
```

</details>

3. Normally, we'd want to use BUSCO to get a rough idea of how complete our genome is. BUSCO (which stands for "Benchmarking Universal Single Copy Orthologs") uses highly conserved single copy orthologs which are expected to be present in every member of a given group. BUSCO is a complicated program with many depencies. Accordingly, it make sense to run BUSCO using Docker. I also frequently encounter problems running BUSCO using Conda. 
    * First, run the following code chunk to write and examine a script called `busco_script.sh`. Don't forget to substitute `[GENOME_NAME]` with the path to your assembled genome
    ```sh
    # echo will text. Using the pipe '>' lets you save the output
    echo 'busco -i [GENOME_NAME] -m genome -l agaricales_odb10 --augustus --augustus_species laccaria_bicolor ' > buscos.sh
    # make script executable
    chmod u+x buscos.sh 
    # make a new subdirectory in the current directory
    mkdir docker_busco
    # copy genome into subdirectory; syntax is cp [source] [dest]
    cp [GENOME_NAME] ./docker_busco
    # copy script into subdirectory
    cp buscos.sh ./docker_busco
    ```
    * We'll then use the BUSCO docker image to run this program: 
    ```sh
    # pull docker image from docker hub 
    docker pull ezlabgva/busco:v5.7.0_cv1
    # run the container created from the image
    docker run -u $(id -u) -it -rm -v ./docker_busco:/busco_wd ezlabgva/busco:v5.7.0_cv1
    # -u specifies your user ID 
    # -it tells the program to run interactively rather than in background 
    # -rm tells the container to stop and shut down upon its completion
    # -v specifies location of a volume (subdirectory which is shared with docker container)
    ```
    * Once your container has been spun-up, simply run BUSCO by running your script: `./buscos.sh`
<details>
<summary><b>Interpretting BUSCO Results</b></summary>

The percentage of complete benchmarking orthologs is often used as a measure of genome completeness when no other information (i.e., a reference genome) is available. However, just because a BUSCO is missing from a genome does not mean that a genome is poor quality. In some circumstances, the gene may be missing as a result of some evolutionary phenomenon. Like many things in biology and statistics, there is no definite BUSCO threshold for determining whether or not a genome is appropriate for a given downstream application. Although the *Pinus taeda* genome is 22 Gb (a typical fungal genome may be around 25 Mb, for comparison), it only has 44% of the BUSCO genes it should. Should this genome be considered useless? **Absolutely not**! Ideally, a genome's BUSCO completeness should be considered along the distribution of read-length statistics, the type of sequencing performed, the distribution of GC-rich sites, and trends in the aforementioned factors in closely related taxa. 

</details>

## Annotation 
<details>
<summary><b>What does it mean to "annotate" a genome?</b></summary>

The assembly process is entirely agnostic to the presence and identity of genes. The process by which genes are located and identified in a genome is called **annotation**. **Gene prediction** is the first step of genomic annotation, and involves inferring the stop sites, start sites, and splice sites of every gene in a given genome, which are then converted into a set of predicted gene sequences. **Functional annotation** is the process by which predicted genes are putatively assigned to functional categories. Many complex algorithms and methodologies are involved in functional annotation! Clustering algorithms (often using reciprocal best-hit BLAST) are used in many programs, as is sequence homology. Other programs use deep-learning algorithms to predict gene function. 

For new bioinformaticians, I recommend using a prepackaged functional annotation suite such as EggNOG. 

</details>

1. Using what you have learned so far, download the program AUGUSTUS using conda. Consult the online manual for AUGUSTUS to determine how to do so. Understanding computational literature is a key part of bioinformatics. If bioinformatics is to play a large part in your future career, you would be best served to get good at reading manuals fast... 
2. Use the EggNOG webserver to provide functional annotations for your gene predictions. Download your results and examine them using Excel. 

## BLASTn
1. Many of you are familiar with BLASTn on NCBI's webservers. BLASTn, BLASTp, tBLASTn, and BLASTx are also available as part of a command-line software package maintained by NCBI. These command line packages allow you to BLAST query sequences against custom databases. Here, our database will be a genome of interest from which we want to extract some gene, which will be our query sequence. Install NCBI's BLAST+ suite in the same manner as above using conda. 

<details>
<Summary>Code</summary>

```sh
conda create -n ncbi-blast bioconda::blast
```
</details>

2. Create a custom BLAST database using `makeblastdb`. If you're unsure how to use this command, try `makeblastdb -h` to bring up the help page. 

<details>
<Summary>Code</summary>

```sh
makeblastdb -in [GENOME_NAME].fastq -dbtype nucl -out AmanitaDB
```

</details>

3. Examine the file `amDODA.fa`. This file contains the DNA sequence for the L-DOPA dioxygenase of *Amanita muscaria*. The deoxygenation of L-DOPA results results in a molecule with reactive aldehyde moieties, allowing for intramolecular reactions that result in the formation of pigment compounds. Use. BLASTn to extract this gene from the provided genome. 

<details>
<summary>Code</summary>

```sh
blastn -db AmanitaDB -out blast_out.tsv -query amDODA.fa -outfmt 6
```

</details>

4. Use what you have learned so far and the resources available to you to determine how you might parse your BLAST output to convert your results to a useable fasta file. 