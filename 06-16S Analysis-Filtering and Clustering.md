

[TOC]

  
## Filtering and Clustering

![](https://i.imgur.com/85h8SCt.png)


Once you have analyzed your sequence quality, you are ready to cluster your sequences. 

We are going to use DADA2 to 
* remove low-quality reads and pairs
* merge
* cluster
* remove chimeras


**What is sequence clustering?**

Grouping similar DNA sequences together – those that are of the same microbial “species”, called amplicon sequence variants (ASVs) or operational taxonomic units (OTUs) in the microbial world

For each group of similar DNA sequences, we are going to get representative sequences so that each amplicon sequence variant (ASV) is represented by a single DNA sequence (file = rep-seqs.qza)

We are also going to use this step to remove low-quality data. The program is going to compute the error frequencies to remove those sequences that are different **as a consequence of** sequencing errors (*But also, do you remember what David explained about singletons?*). Finally, we will also remove “junk” sequences (such as chimeric sequences and low quality). 
Chimeric sequence = combination of multiple erroneously joined DNA sequences (can be an artifact of PCR amplification). 
Singleton = a single, unique DNA sequence. Some of them could be considered a sequencing artifact. 

After sequence clustering, you will have a table of samples by microbial species, with abundances (#number of each type of bacterial sequence in each sample). This information is contained in a file called "table.qza"


Now, you are ready to run DADA2. Note that this step takes the longest because it is very computationally intensive... it can take many hours to run, depending on how much data you have! **(so again, are you running in screen?)**

more info: here there is a link to the R package of dada2: https://benjjneb.github.io/dada2/tutorial.html



## Run Dada2 
(this command will take a while to finish) 

The input file contains all the reads 'paired-end-demux.qza'
The region v4 275 nt in length. We have reads that are 300 nt. We are going to trim each read (usually the tails of the reads have the lowest quality) maintaining an overlap between the reads of around 20 nt. We are also trimming at the 5' end, the beginning of the sequence.  

for more info see here: https://docs.qiime2.org/2020.6/plugins/available/dada2/denoise-paired/ 

:::success
qiime dada2 denoise-paired \
&nbsp;    --i-demultiplexed-seqs paired-end-demux.qza \
&nbsp;    -–p-trim-left-f 12 \
&nbsp;    --p-trim-left-r 12\
&nbsp;    --p-trunc-len-f 240 \
&nbsp;    --p-trunc-len-r 240 \
&nbsp;    --o-table table.qza \
&nbsp;    **--p-n-threads 16** \
&nbsp;    --o-representative-sequences rep-seqs.qza \
&nbsp;    --o-denoising-stats denoising-stats.qza \
&nbsp;    --verbose\
:::

Run in the terminal

```
qiime dada2 denoise-paired  --i-demultiplexed-seqs paired-end-demux.qza  --p-trim-left-f 12  --p-trim-left-r 12 --p-trunc-len-f 240  --p-trunc-len-r 240  --o-table table.qza  --p-n-threads 16 --o-representative-sequences rep-seqs.qza   --o-denoising-stats denoising-stats.qza --verbose

```


## Summarize the results

Next, create summary tables of clustered sequences.

:::success
qiime feature-table summarize  \
&nbsp;    --i-table table.qza \
&nbsp;    --o-visualization table.qzv \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::
Run in the terminal

```
qiime feature-table summarize  --i-table table.qza --o-visualization table.qzv --m-sample-metadata-file sample-metadata.tsv
```
:::success
qiime feature-table tabulate-seqs \
&nbsp;    --i-data rep-seqs.qza  \
&nbsp;    --o-visualization rep-seqs.qzv
:::
Run in the terminal
```
qiime feature-table tabulate-seqs --i-data rep-seqs.qza  --o-visualization rep-seqs.qzv
```
**Check the stats table, how many reads were removed because of low quality** This directly depends on the parameters you set to run the program so if too many reads are removed, you might want to double-check those parameters. Typically around 40-50% of reads are retained. 

:::success
qiime metadata tabulate   \
&nbsp;    --m-input-file denoising-stats.qza  \
&nbsp;    --o-visualization denoising-stats.qzv
:::
Run in the terminal
```
qiime metadata tabulate   --m-input-file denoising-stats.qza  --o-visualization denoising-stats.qzv

```
Look at the “Overview” tab. How many samples do you have? How many unique sequences are there? How many total sequences? 

Go to the “Interactive Sample Detail” tab and view the number of sequences (“features”) in each sample. Use the sliding bar to see how many samples you will retain at each sampling depth. Do you need to exclude any samples?

Look carefully at the denoising stats. If too many seqs are lost and the quality of the reads does not indicate major issues you might need to adjust the dada2 parameters. Are you truncating the sequences too much? too little?






![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 


