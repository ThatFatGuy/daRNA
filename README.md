# daRNA
Secondary RNAseq analysis - testing

**There are two different options for this - 1) Using raw files for quant, 2) using already made bam files for quant.**

>**Option 1**

>Assuming you have trimmed 'raw' reads

```bash
export PATH=/Volumes/BiochemXsan/staff_groups/brownlab/SJTW/Salmon-0.7.2_linux_x86_64/bin/:$PATH

```
>The above code simply tells your session (you have to do it every time you start a new session - ONCE per session) - where to find salmon, in this case i have a version installing in my directory in Breezy's group, this SHOULD work, but if it doesnt then there might be some weird permission stuff going on and i can move it.

>**Next you will need to download the fasta file for all the genes in PAO1 - just get this from pseudomonas.com (the ORF_DNA file), move it to a sensible directory on the server - lets just say you move it to "scratch/lamontlab/jess", which is where most of your other analysis stuff is**

```bash
cd /Volumes/BiochemXsan/scratch/lamontlab/jess

salmon index -t PAO1genes.fna -i PAO1index

salmon quant -p 20 -i PAO1index -l A -1 trimomatic/C7N92ANXX-1235-07-21-1_L007_R1_Trim.gz -2 trimomatic/C7N92ANXX-1235-07-21-1_L007_R2_Trim.gz -o sampleXquant --numBootstraps 100

```
>**The above code - 1) will move you to the directory containing the file you downloaded from pseudomonas.com (provided you moved it to your folder on scratch) 2) indexes the file from pseudomonas.com - this puts it into another folder, make sure when you do step three you DONT write -t PAO1index/ make sure its -t PAO1index  3) this is taking sample X raw reads (trimmed) and is using the entire server to map it across and count it in about 3mins (with 100 bootstraps).**

>If you are interested in the result straight away you can look at the "quant.sf" in the output folder from salmon, it can be opened in excel and just has a gene list with counts and TPM.

>**Option 2 (note - i have not tested this yet)**

>DO EVERYTHING ABOVE EXCEPT - the final command with salmon quant, this is the only changed part.

```bash
salmon quant -p 20 -t PAO1genes.fna -l A -a trimmomatic/aligned/sampleX.bam.bam -o sampleXquant --numBootstraps 100

```

>This should take the bam files which you made in the past (i assume they are correct) and then count them in regards to the raw fasta file from pseudomonas.com (does not use and index this time). I have no idea if this is faster or more accurate - in theory it should be both. It will also be quicker for you, because the BAM files already exist and then it allows for a direct comparison of what you have already done because these are the same BAM files you used for that.

**NOW FOR THE HARD PART - USING R**

>If you open chrome/firefox/edge(heaven forbin) and go to **biocrstudio:8787** this is the departments R server, its super handy because its connected to the server (with all your files) and is really quick - log in how you would on the server. You can also do whatever you want on it and install things at will, it will also save exactly what you were doing even if you accidentally close your browser.

```R
 
source("http://bioconductor.org/biocLite.R")

biocLite("devtools")   

devtools::install_github("pachterlab/sleuth")

devtools::install_github("COMBINE-lab/wasabi")

library('seluth')

library('wasabi')

```
>This will install all the packages needed - it might ask if you want to overwrite stuff and give you the option for all/some/none (a/s/n) just press a.

>**Now you need to change your working directory to where all your files are (the default working directory is your student directory on the server - so you can move ALL the quant folders across or you can move change it in R**

```R
setwd("/Volumes/BiochemXsan/scratch/lamontlab/jess")

```
>Right now for some fun/mildly infuriating stuff, you need to prepare the salmon outputs for slueth so to do that

```R
prepare_fish_for_sleuth("outputXquant")
prepare_fish_for_sleuth("outputYquant")
prepare_fish_for_sleuth("outputZquant")

```
>Once all of the samples have been converted then you need to make a file that tells sleuth what your experimental design was, lets just (for this example) assume you have done ALL of your samples and want to look at all of them at once (seeing as this would be the hardest to do so its the easiest for me to show you). **lets also assume you have them in order of control (T0), saline then 5-FC, this will make things easier for me but realistically this doesnt affect you at all**

```R
genotypes <- c(rep('control_t0', 3), rep('saline', 9), rep('5-FC, 9)

```
>This makes a file called genotypes that has a list that contain control_t0 three times and the others 9 times (all the samples in your analysis) - this needs to be changed if you have different samples or different numbers.

>**Next you need to give more details about the experiment by making another file**

```R
conD01 <- data.frame(
  sample = rep(c('1 t0', '2 t0', '3 t0, '1 t10min car', '1 t60min car', '1 t4h car', '2 t10min car', and so on and on)
  genotype = genotypes
  growth = rep(c(':control', ':control', ':control', ':saline', ':saline', ':saline', ':saline', and so on and on)
  treatment = rep(c(':control', ':control', ':control', ':no', ':no', ':no', ':no', and so on and on)
  time = rep(c(':0', ':0', ':0', ':10', ':60', ':240', ':10', ':60', and so on you get the idea by this point)
path = rep(c('sampleX', 'sampleY', 'sampleZ' and so on and on))

conD01$path <- as.character(conD01$path)

```
**MAKE SURE that "path" is actually linking to the path of the files you prepared for sleuth (because you are in the right directory it should just need you to list the folder name) - make sure everything is in the same order, genotypes link up with sample name and growth and treatment etc. Because the conD1 (condition1) file will be used to do all the stats from - this part will take some time to make so just be patient and triple check it.**

```R
#now you need to tell the program what you are actually interested in - ie : treatment

model <- "~ treatment"

#Then compare the data to the model you want to test (note you could also put time or growth etc)

mytest01 <- sleuth_prep(conD01, as.formula(model)) %>%
            sleuth_fit()
models(mytest01)

#then finally

mytest01 <- sleuth_wt(mytest01, which_beta = 'treatment:yes'

sleuth_live(mytest01)

```

**That simple (hahaha), just remember you can change what you are asking the program by changing the model from treatment to time or growth etc, just remember to also change the sleuth_wt part accordingly**

**hopefully this works for you, let me know how it goes**


  






