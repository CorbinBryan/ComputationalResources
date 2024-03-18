<details>
<summary>Quality Control</summary>

Your first step in assembling a genome is to remove poor quality base calls and adaptor sequences from your raw DNA reads. To do so, we'll use a program called Trimmomatic, which can be easily installed using Conda. 

</details> 

1. We'll first create a new conda environment called `trim` and download trimmomatic in that environment
```sh
# create a new environment with name trim 
conda create -n trim 
# make trim the active environment (env to which pkgs installed)
conda activate trim 
# install trimmomatic from bioconda channel 
conda install -c bioconda trimmomatic 
```
