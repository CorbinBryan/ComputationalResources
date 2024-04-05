# Strategy I 
This attempt involves two different data sets:
1. Jut the data from (Geml et al., 2008)
2. Combined data from (Geml et al., 2008) and a [previous personal project](https://github.com/CorbinBryan/SectAmanitaPhylo).  

Note that there are multiple runs for each set. 
## Geml data runs  
### Obtaining & Aligning Data, Trimming alignments.
1. Retrieve all `.fasta` files (one per locus) from GenBank. This entire population set is available on GenBank and can be directly downloaded. 
2. Run `align_trim.sh`, which aligns each locus and trims the resulting alignment. 
3. Load all alignments in `./alignments` into BEAUti. Under the "guess" popup, click the regular expression option and use either `^(?:[^_\n]+_){4}([0-9A-Za-z]+)(?:_[^_\n]+)*$` or `^(?:[^_\n]+_){5}([0-9A-Za-z]+)(?:_[^_\n]+)*$`. Manually edit the rest by hand.
---
---
**Alignment Information:** MAFFT: L-INSi algorithm with a maximum of 1000 iterations and automatic direction adjustment. (Most accurate algorithm, no substantial reduction in run-time compared to the fast algorithm for a data set of this size).  
 
**trimAl Information**: Run with the default settings, which automatically selects and adjusts the trimming algorithm given the dataset. 

---
---
### *First Run*
* **Relevant Model Information:**  
  * Substition model: TN93+G4  
  * Estimate gamma rate value and shape hyperparameter. Estimate nucleotide frequences and both parameters required by TN93 ($\kappa_1, \kappa_2$). Do note estimate clock rates for any locus, nor for the species tree. Assume a Yule model as the tree prior, which requires one less parameter than the BD Model, and runs faster accordingly. By default, the working directory is set to the `./alignments`. See `muscaria.xml` under `./alignments`.

First run was a failure due to vanishingly low support values. It also took far longer to run. Turns out several taxa were still left as 'strain'. This was despite being run for 30 million iterations (sampling every 5,000). 

### *Second Run*
* As before. Yule changed out for a birth death model. Otherwise, priors the saem. Run time is noticibly faster (1/3 the time for 1 million MCMC iterations). Also edited several taxa still listed as 'strain'. The algorithm forces taxa of the same name to coalesce, which may have resulted in the poor results observed above. 
* Support values were much higher but not particularly good. Notably, the tracer plot showed excellent mixing and convergence, suggesting that 30 million iterations was sufficient or more than sufficient for this data set. See `Tracers_Trees.Rmd` in `./R`

### *Third Run* 
* Previous runs showed poor support values--perhaps worse than Geml et al and worse than my previous analysis. To improve this process, I'll be combining this data with my own previous data set from a previous project examining the phylogeny of sect. *Amanita*. 


## Combined Runs
### Preparing the Combined Dataset: 
1. Remove gaps with `sed 's/-//g'`. This will leave a lot of empty lines. Just remove these by hand. Gapless sequences from the SectAmanitaPhylo project are shown in `./raw_fastas`, denoted by `[LOCUS]2.fa`. The fastas with the prefix `update_` in that directory are the raw fastas without the gaps removed. 
2. Construct preliminary alignments, trim them, and construct preliminary trees. Remove one sequence from ITS. Should be easy to determine which sequence is removed later. 
3. Remove duplicates with `rm_dup.sh`. It takes the name of a file in `./comb_al` as input. The output files will be put into wherever the script is run from, so run the script in the root of this directory.
4. Realign resulting files (`c*_nodups.fa`). No need to retrim. Put alignments in `./comb_al_nodup`, saving each as `c[LOCUS].fa`. After all alignments have been generated, perform `rm *_nodups.fa`.
5. Run `redo_names.sh`, which takes the name of a file and outputs a file with suffix `_al_fin.fa` in the working directory. These will `.fasta`s should be read for down stream analysis with starBEAST3. Move all `.fastas` into `./stb3`. It is now ready. 
6. Run starBEAST3 as before. 
### _First Combined Run_ 
* LG382, LG458, LG862, LG864, LG882, LG1045, LG1066 appear to be interferring with the phylogeny. They may not even be in this clade, or may be particularly early diverging. These have been removed. 
* See `./R/Tracers_Trees.Rmd` for the tracer file. It appeared to look relatively good, all things considered. I'm going to run just the ITS tree with iqtree2 to see if it looks any better. This was done with the following: `iqtree2 -s cITS_al_fin.fa -alrt 1000`. Notice that I decided to use the SH-aLRT as a support value. It may be prudent to use model finder on each locus prior to running a Bayesian analysis. ModelFinder suggests HKY+F+I+R2 for use with ITS.
> *Note:* The F in the above model implies that base frequencces are empiricle rather than inferred using ML. The I above allows for a portion of invariable sites. The R refers to a free rate model, which supposedly often fits data better than a gamma model due to its relaxed assumptions. 
* In an effort to improve Bayesian posterior values, I'll check the others with model finder as well. Model finder suggests TN+F+R2 for the LSU alignment. ModelFinder suggests K2P+G4 for BTUB. Lastly, it suggests TNe+G4 for the EF1-alpha locus. 
* It is generally considered bad practice to mix Bayesian and ML paradigms in the fashion I just did. However, Model Finder did suggest all of these based on the Bayesian Information Criterion (BIC), but I'm not exactly sure how that operates.
* Looked like trash, going to rerun
### _Second Combined Run_ 
* I removed _A. subfrostiana_ sequences, as they appear to be causing trouble, likely due to incomplete taxa sampling. 
* I think I may have forgotten to re-align the sequences. Realign, then trim as before. I moved the `*_al_fin.fa` files into a subdirectory of `./stb3`. 
* Run was not much better. Estimating the additional parameter caused problems. 
### _Third Combined Run_
* Rerunning with TN93+G4 for EF1 and BTUB, and TN93+G4+F for LSU, ITS. 
* results poor, again, though mixing was pretty great. 
  
# Strategy 2 
Previous attempts, even with the combined set, yielded poor support values. As such, I'm redoing it one last time with an updated set that features better sampling across taxa. All unmodified sequence files have been moved to `./Raw_Data`, though their respective subdirectories remain the same. Likewise, all modified sequences (i.e., alignments, etc) have been moved to `./Derived_Data`. 
## Obtaining Sequences 
* See files with `_acc_no` suffix in `./expanded_run` for information on the loci available in the new data set. 
* All aspects of this part of the analysis will take place in `./expanded_run`. 
* Download GB accessions with the following: 
```{sh}
while read acc; do 
  efetch -db nucleotide -id "$acc" -format fasta >> [output]
done < [locus]_acc_no 
```
* While downloading the EF1 sequences, several failed for some reason. All that failed were from the KR series. 
* While downloading RBP2 sequences one failed. 
* When done, moved all acc_no files into their own directory in `expanded_run`. 
* During a test run of just the ITS locus, I found a misidentified beta-tubulin sequence, which I removed. I also found that I had inadvertently included several accessions multiple times. These were removed. 
* Per usual, several ITS accessions had to be removed from the final set on account of some relatively large in-dels that obscured their relationships. Also, partial seqs.  
* I checked all of the accessions by aligning with MAFFT (default settings, direction adjustment activated), trimming the alignments with trimal, and inferring the maximum likelihood phylogeny using IQTree2. I then removed sequences from the original fasta that were obviously either misidentified or contained uncommon indels that resulted in unbelievably long branch lengths. For the most part, these sequences appear to be simply misidentified.
## Aligining, Trimming, Prepping for Inference
* Run the following, in the `./expanded_run` directory: 
```{sh}
for locus in $(ls ./raw_fa); do
  pref=$(echo "$locus" | sed 's/.fa//g')
  mafft-linsi --adjustdirection ./raw_fa/${locus} | sed 's/ /_/g' > ./al_fa/${pref}_al.fa
done 
```
* Don't forget to double check that there are no spaces in the fasta sequence headers. Trim the alignments with trimal using the following: 
```{sh}
LOCI=("ITS" "LSU" "EF1" "BTUB" "RBP")

for locus in ${LOCI[@]}; do 
  trimal -in ./al_fa/${locus}_al.fa -out ./trim_al_fa/${locus}_tral.fa
done 
```
* There are view sequences in this expanded run that do not include species level designations. All of these have isolate codes. I think it would be better to force these to coalesce as a single species. To do so, I'm going to once again generate and ML tree (with IQTree2) and see where all of the isolates (nearly all being from (Geml et al., 2008)). This time, I've included SH-aLRT support values with 1000 iterations as a measure of support. Once again, this branch support value is less conservative than traditional bootstrapping. Note that aLRT stands for approximate likelihood ratio test, and SH stands for Shimodaira–Hasegawa, who developed an algorithm for comparing likelihoods between trees. More information on the test can be found [here](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3158332/). 
* Actually, I just remembered that I am able to just check on RET's website. Do that and edit by hand. Realign these and put the alignments in `alter_trim_al`.  
* While waiting for BEAST to run, I tried using ASTRAL. The results were absolute garbage and certainly not worth the time of running it. 
### _First Run Results_
* Tracer plots reveal mixing was not ideal, but was not necessarily poor either. Convergence was good, however. Ultimately, it might be better to run this again with 15 million or 20 million iterations (fits well with previous runs). 
* Posterior supports are generally quite good. 
* This was once again done with a Yule tree prior, TN93+G4 as the substitution model (two parameters is sufficient or more than sufficient). 
### _Second Run Results_
* tracing good but pseudogemmata causing issues. Remove it's ITS sequence, rerun after realigning. 
* Note: guessowii still closer to A. muscaria var. muscaria than to flavivolvata... 
## Relaxed Clock Analysis 
### Set Up
* copy `./expanded_run/ready_for_sb3` into the main directory. This will allow for easier transfer of files to and from the CHTC. 
* HTCondor submission file is available at `./ready_for_sb3/sb3.sub`
* Note that `biocontainers/beast2-mcmc` is the `docker_image` specified for this run. This is much easier than attempting to use conda, because most commonly used bioinformatics packages already have a `biocontainers` image. 
* First, I'm going to install beast2 with bioconda. This must be done in it's own directory, likely owing to the fact that beast2 requires a different version of BEAGLE than other packages I've used. 
* Note: I had to uninstall all of miniconda on the CHTC and reinstall it. 
* Note that you must incldue `packagemanager -add starbeast3` in the script you use. 
* I took a break from working on that and made a script (`musc_names.sh`) to rename the _A. muscaria_ var. _muscaria_ collections previous labeled with strain and collection numbers. I previously edited these by hand. I renamed the files to have the following suffix: `_load.fa`. 
* These aligned fastas were then loaded into BEAUti. This time, however, I decided to estimate the relative partition rates and assumed a relaxed clock model. 
> **Relaxed Clock Models**:  
> Relaxed clock models assume that each branch of a species tree has an indepedent and identically distributed substition rate according to some user specified distribution. In otherwords $\mu\sim \text{Lognormal}(1,\sigma^2)$, where $\sigma^2$ is the standard deviation of the lognormal distribution, which in this case has mean one. This value is calculable from the data. 
* You're going to do the following, locally: 
`docker build --platform linux/x86_64 -t multimuscaria/beast2:v15 - < Dockerfile` (must be done in dir. with docker file)
* Docker file is in the root of this directory. 
* I made a publically available docker image containing beast2. 
* Note: let's set LSU clock rate to one. This is advisable when there are few loci. 
* Forgot to mention above.
* Tried to run using docker, but instead ended up doing it locally. 30 million iterations of the MCMC. Tried with three burn in percentaged: 10, 30, and 50% burn in removed. 
* Apparently it's not a good idea to estimate substitution rate and clock rate for gene trees, because this make it unidentifiable. I'm going to run it one last time with this updated data set (i.e., with _A. psuedogemmata_ removed). 
* We're going to run it one last time with 20 million iterations, everything the same except that **no** partition clock rates will be estimated and the species tree will be left with a strict clock (see next section.)
## Strict Clock Analysis 
* working directory for this analysis is `./sb3_final`. The `.xml` file for this analysis is `sb3_final.xml`. 
* 20 million iterations. 
* Strict clock, no partition clock rates estimated. 