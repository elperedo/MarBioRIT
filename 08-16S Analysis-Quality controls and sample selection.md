[TOC]


# Quality controls and sample selection

## Rarefaction Analysis 

**Now that we have a bacterial dataset, we will examine how thoroughly we have "sampled our sample"**

Our original DNA extractions represent the population of microbes in the environment. We know that different taxa are present in different abundances in any sample. If our sequencing effort has not been enough, we might be missing out on those taxa that are not very abundant (rare biosphere). How do we know id we may or may not have sequenced our samples deeply enough to detect everything that is in the sample?
We test whether we have reached saturation in taxa discovery, that is, we have identified (most of) the organisms in that sample. 
 
## Rarefaction curve for visualization of taxa discovery. 

We are going to visualize whether your taxa discovery for each sample has reached saturation given your sequencing effort using a rarefaction curve. We will also be able to determine at what sequencing depth saturation happened for each sample. 
The alpha rarefaction command randomly subsamples your dataset at n = 1000, n = 2000, n = 3000... n = 50,000 sequences, and it plots the # of unique microbial taxa at each sequencing depth, generating a rarefaction curve. 

*Here we will use a --p-max-depth of 65000. If this is too high, the program will tell you that the maximum number of features is **X**. Use that number as --p-max-depth instead.*

:::success
qiime diversity alpha-rarefaction  \
&nbsp; --i-table table-no-mitochondria-no-chloroplast.qza \
&nbsp; --p-max-depth 65000  \
&nbsp; --m-metadata-file sample-metadata.tsv  \
&nbsp; --o-visualization alpha-rarefaction.qzv
:::

```
qiime diversity alpha-rarefaction   --i-table table-no-mitochondria-no-chloroplast.qza --p-max-depth 65000  --m-metadata-file sample-metadata.tsv  --o-visualization alpha-rarefaction.qzv

```
you can also run a second curve with a smaller sequencing depth to focus on the beginning of the curve

```
qiime diversity alpha-rarefaction   --i-table table-no-mitochondria-no-chloroplast.qza --p-max-depth 8000  --m-metadata-file sample-metadata.tsv  --o-visualization alpha-rarefaction-2.qzv
```

If your visualization looks something like this then all samples were sequenced to a depth sufficient to sample all (or almost all) of the diversity:

![](https://i.imgur.com/KT8kR6u.png)

If you have a sample that is not reaching or approaching an asymptote then you (1) could consider investing in deeper sequencing of the sample or (2) you can remove the under-sequenced sample. 
You don't need to reanalyze all our data, we can remove a sample by filtering it out using the metadata file (next section). 

Now, use the graphs and table files to check if all your samples have reached saturation. If not, identify those samples that should be filtered out from the analysis.

![](https://i.imgur.com/osCopOU.png)

![](https://i.imgur.com/bJSX5K2.png)


## Metadata-based filtering

You might want to focus at this time on a subset of samples or remove those samples that are low quality. To do so, we can create new files and only include the samples that you want to analyze. To do this, we are going to filter samples. 

To filter samples based on quality: 
One of the easiest ways of doing this is to modify the metadata file by adding a column called `keep`. Identify the samples you want to keep with the category "yes", and those to be filtered with "no".


![](https://i.imgur.com/WJRoTO5.png)



`--p-where "[keep]='yes'"`


**From this point onwards, "table-no-mitochondria-no-chloroplast.qza" becomes "table-filtered-no-mitochondria-no-chloroplast.qza"!and only includes the subset of samples of interest**

:::success
qiime feature-table filter-samples  \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --m-metadata-file sample-metadata.tsv   \
&nbsp;    --p-where "[keep]='yes'"   \
&nbsp;    --o-filtered-table table-filtered-no-mitochondria-no-chloroplast.qza
    
:::

```
qiime feature-table filter-samples  --i-table table-no-mitochondria-no-chloroplast.qza  --m-metadata-file sample-metadata.tsv   --p-where "[keep]='yes'"  --o-filtered-table table-filtered-no-mitochondria-no-chloroplast.qza

```

Now let's visualize the new filtered table
We will use the same scripts we used after dada2, just modifying the input table to create summary tables of sequences.

Type:


:::success
qiime feature-table summarize   \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --o-visualization table-filtered-no-mitochondria-no-chloroplast.qzv   \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::

```
qiime feature-table summarize  --i-table table-filtered-no-mitochondria-no-chloroplast.qza  --o-visualization table-filtered-no-mitochondria-no-chloroplast.qzv   --m-sample-metadata-file sample-metadata.tsv
```

:::warning
** How to filter samples based on metadata information:**
You can select your samples based on any of the columns in the metadata. 
Maybe you only want the DNA samples or the samples from the initial time point. 
You can use the same approach, just select the column from the metadata file and the variable. 
```
qiime feature-table filter-samples  \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --m-metadata-file sample-metadata.tsv   \
&nbsp;    --p-where "[ColumnName]='ANYTHING_YOU_WANT'"  \
&nbsp;    --o-filtered-table table-ANYTHING_YOU_WANT-no-mitochondria-no-chloroplast.qza
 ```   

**If multiple values should be retained from a single metadata column, the IN clause can be used to specify those values.** For example, the following command can be used to retain all samples with Ulva and Seawater.

```
qiime feature-table filter-samples    \
&nbsp;      --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;      --m-metadata-file sample-metadata.tsv   \
&nbsp;      --p-where "[ColumnName] IN ('ANYTHING_YOU_WANT01','ANYTHING_YOU_WANT02')"   \
&nbsp;      --o-filtered-table table-ANYTHING_YOU_WANT_01_02-no-mitochondria-no-chloroplast.qza

```
:::


# Generate final taxonomy graphs

Use the final filtered files to generate the bar plots. 
:::danger
If you have filtered by metadata make sure you are calling the right files. Double-check the table file names. In fact, for your convenience, I recommend you rename the table files from `table-ANYTHING_YOU_WANT_01_02-no-mitochondria-no-chloroplast.qza` to `table-filtered-no-mitochondria-no-chloroplast.qza` as this is how the file will be called from now on.

in you didnt filter anything, you can make a copy and rename the file

```
cp table-no-mitochondria-no-chloroplast.qza table-filtered-no-mitochondria-no-chloroplast.qza
```
:::

 
 :::success
 qiime taxa barplot   \
 &nbsp; --i-table table-filtered-no-mitochondria-no-chloroplast.qza  \
 &nbsp; --i-taxonomy taxonomy.qza   \
 &nbsp; --m-metadata-file sample-metadata.tsv  \
 &nbsp; --o-visualization taxa-bar-plots-filtered-no-mitochondria-no-chloroplast.qzv
 :::
 
 ``` 
 qiime taxa barplot   --i-table table-filtered-no-mitochondria-no-chloroplast.qza   --i-taxonomy taxonomy.qza   --m-metadata-file sample-metadata.tsv   --o-visualization taxa-bar-plots-filtered-no-mitochondria-no-chloroplast.qzv
 
 ```
 
 :::info
 **Can you see this graph?**
 What is the impact of the different treatments on the samples?
Does it matter if we look at who is present (DNA) or who is active (RNA)?

 
 ![](https://i.imgur.com/VpoxUeh.png)
 :::
 
**Export as ASV table**

ASV tables are useful for additional analyses, e.g. for differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-filtered-no-mitochondria-no-chloroplast.qza  --output-path ASV-table-filtered-no-mitochondria-no-chloroplast
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-filtered-no-mitochondria-no-chloroplast/feature-table.biom -o ASV-table-filtered-no-mitochondria-no-chloroplast/feature-table.tsv --to-tsv
```

## Taxonomy-collapse 
Now that we have a definitive table without eukaryotic sequences and filtered low-quality samples, we can use this option to export data at the taxonomical rank of our choice. This simplified our dataset which is useful for visualization but also in some of the additional analyses we will run.  

https://docs.qiime2.org/2021.8/tutorials/pd-mice/taxonomic-classification-again

```
qiime taxa collapse  --i-table table-filtered-no-mitochondria-no-chloroplast.qza --i-taxonomy taxonomy.qza   --o-collapsed-table table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza --p-level 7
```
You can export the table into excel for additional analysis.  
```
qiime tools export   --input-path table-collapsed-filtered-no-mitochondria-no-chloroplast-level7.qza  --output-path table-collapsed-filtered-no-mitochondria-no-chloroplast-level7
```

Convert the exported table from biom format to tsv
```
biom convert -i table-collapsed-filtered-no-mitochondria-no-chloroplast-level7/feature-table.biom -o table-collapsed-filtered-no-mitochondria-no-chloroplast-level7/feature-table.tsv --to-tsv
```

![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 

