## Part I. Statistical Analysis

## Beta Diversity Analyses
 
You've probably already noticed that the number of reads is quite different across samples.  Analyses of beta diversity behave very oddly if there are large differences in the number of reads across the compared samples.  Thus many programs, including QIIME, require an equal number of reads across samples.  There are two different methods available in QIIME2 to do this: normalizing by "rarefying," or throwing away sequences from samples until all samples have the same number of reads, or by converting the counts in each sample to frequencies.  

* **Rarefy**: subsample the same number of sequences from each sample
* **Relative Frequency**: proportion of each feature out of the total feature counts for each sample


### Normalizing by Rarefaction

Based on the results of your rarefaction analysis, choose a sampling depth that each sample should be reduced.  Samples, where the sum of frequencies is less than the sampling depth, will not be included in the resulting table unless subsampling is performed with replacement.  Here we are randomly sampling 20,000 sequences per sample:

:::success
qiime feature-table rarefy \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast.qza  \
&nbsp;    --p-sampling-depth 20000 \
&nbsp; --p-with-replacement    \
&nbsp;    --o-rarefied-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza
:::

```
qiime feature-table rarefy  --i-table table-filtered-no-mitochondria-no-chloroplast.qza --p-sampling-depth 20000 --p-with-replacement --o-rarefied-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza
```

:::success
qiime feature-table summarize  \
&nbsp;    --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza  \
&nbsp;    --o-visualization table-filtered-no-mitochondria-no-chloroplast-rarefied.qzv  \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::

```
qiime feature-table summarize  --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza  --o-visualization table-filtered-no-mitochondria-no-chloroplast-rarefied.qzv --m-sample-metadata-file sample-metadata.tsv
```

**Export an ASV table**

ASV tables are useful for additional analyses, e.g. for differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-filtered-no-mitochondria-no-chloroplast-rarefied.qza  --output-path ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied/feature-table.biom -o ASV-table-filtered-no-mitochondria-no-chloroplast-rarefied/feature-table.tsv --to-tsv
```

## Calculating non-phylogenetic metrics

Here we are going to calculate the beta diversity using Normalization by rarefication.
As with alpha diversity, there are many ways to express the beta diversity of two samples based on the presence/absence and/or relative abundances of taxa (OTUs, ASVs). The [Jaccard Index](https://en.wikipedia.org/wiki/Jaccard_index) measures similarity based on presence/absence; [Bray-Curtis](https://en.wikipedia.org/wiki/Bray%E2%80%93Curtis_dissimilarity) is a common method that combines presence/absence with relative abundance. For a full list of available metrics:
```
qiime diversity beta --help
```
:::info
[More info on alpha and beta diversity metrics](https://forum.qiime2.org/t/alpha-and-beta-diversity-explanations-and-commands/2282)
:::

We will use the 'table-filtered-no-mitochondria-no-chloroplast-**frequency**.qza' as input to beta diversity analyses, which are set up in the same manner as the alpha diversity analyses above.  Using the frequency normalized table and Jaccard:
:::success
qiime diversity beta \
&nbsp;   --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza \
&nbsp;   --p-metric jaccard  \
&nbsp;   --o-distance-matrix BETA-jaccard-distance-matrix.qza
:::
```
qiime diversity beta --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza --p-metric jaccard --o-distance-matrix BETA-jaccard-distance-matrix.qza
```
This will assess the significance of all pairwise combinations and across all groups using permanova:
:::success
qiime diversity beta-group-significance \
&nbsp;   --i-distance-matrix BETA-jaccard_distance_matrix.qza \
&nbsp;   --m-metadata-file sample-metadata.tsv \
&nbsp;   --m-metadata-column [column of data that you want to evaluate]\
&nbsp;   --p-method permanova  \
&nbsp;   --p-pairwise \
&nbsp;   --o-visualization BETA-jaccard-group-significance-[remember to edit name to prevent overwriting].qzv
:::
```
qiime diversity beta-group-significance --i-distance-matrix BETA-jaccard-distance-matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column Sulfide_Treatment_nucleic_acid_time_point --p-method permanova --p-pairwise --o-visualization BETA-jaccard-group-significance-Sulfide_Treatment_nucleic_acid_time_point.qzv
```
:::info
Can you explain these results?

![](https://i.imgur.com/B2uhxIj.png)
:::



Now, run the analysis for testing whether the present (DNA) and active (RNA) microbial communities are identical, using Jaccard and at least another different metric. You can change the metric by changing the `p-metric` parameter.


##  Calculating phylogenetic metrics (Unifrac)

### Create a tree

[Uni frac](https://en.wikipedia.org/wiki/UniFrac) UniFrac is a distance metric used for comparing biological communities. It differs from beta metrics such as Bray-Curtis in that it incorporates phylogenetic distances between observed organisms in the computation. Both weighted (quantitative) and unweighted (qualitative) variants of UniFrac are widely used in microbial ecology, where the former accounts for the abundance of observed organisms, while the latter only considers their presence or absence. The method was devised by Catherine Lozupone when she was working under Rob Knight of the University of Colorado at Boulder in 2005.

First, we group the sequences using a phylogenetic tree. Trees can be "rooted" or unrooted." Rooted trees contain a root at the base of the tree, which represents the common ancestor to all of the branches, while unrooted trees display relationships among taxa without assumptions about common ancestry.  Most phylogenetic methods produce unrooted trees, but these are often rooted at the midpoint for convenience.
![](https://i.imgur.com/qidd35j.png)

There are many methods to build these trees-- each starts by aligning sequences so that homologous positions can be compared, then using an evolutionary model to infer how the sequences are related.  We will use the `phylogeny` tool in QIIME to align sequences using the [MAFFT](https://mafft.cbrc.jp/alignment/software/) algorithm and build rooted and unrooted trees using the approximate likelihood method of [fasttree](http://www.microbesonline.org/fasttree/): 
:::success
qiime phylogeny align-to-tree-mafft-fasttree \
&nbsp;   --i-sequences rep-seqs.qza  \
&nbsp;   --o-alignment aligned-rep-seqs.qza  \
&nbsp;   --o-masked-alignment masked-aligned-rep-seqs.qza \
&nbsp;   --o-tree unrooted-tree.qza  \
&nbsp;   --o-rooted-tree rooted-tree.qza 
&nbsp;   --p-n-threads 4
:::
```
qiime phylogeny align-to-tree-mafft-fasttree --i-sequences rep-seqs.qza --o-alignment aligned-rep-seqs.qza --o-masked-alignment masked-aligned-rep-seqs.qza --o-tree unrooted-tree.qza  --o-rooted-tree rooted-tree.qza --p-n-threads 4
```


You can visualize the tree in the interactive tree of the live website (Itol)

Upload your tree.qza here and drag and drop into the tree taxonomy.qza and table.qza. 
https://itol.embl.de/upload.cgi

![](https://i.imgur.com/ZXM2iV2.png)



### Weighted and unweighted unifrac
Now that we have a phylogeny, we can use Unifrac metrics. These metrics take into consideration the phylogenetic relationships in the community. 


This example is for unweighted Unifrac:
:::success
qiime diversity beta-phylogenetic \
&nbsp;   --i-table filtered-table-no-mitochondria-no-chloroplast-rarefied.qza \
&nbsp;   --i-phylogeny rooted-tree.qza \
&nbsp;   --p-metric unweighted_unifrac \
&nbsp;   --o-distance-matrix BETA-unweighted-unifrac-distance-matrix.qza
:::

```
qiime diversity beta-phylogenetic --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza --i-phylogeny rooted-tree.qza --p-metric unweighted_unifrac --o-distance-matrix BETA-unweighted-unifrac-distance-matrix.qza
```


This will assess the significance of all pairwise combinations and across all groups using permanova:
:::success
qiime diversity beta-group-significance \
&nbsp;   --i-distance-matrix BETA-unweighted-unifrac-distance-matrix.qza \
&nbsp;   --m-metadata-file sample-metadata.tsv \
&nbsp;   --m-metadata-column [column of data that you want to evaluate]\
&nbsp;   --p-method permanova  \
&nbsp;   --p-pairwise \
&nbsp;   --o-visualization BETA-unweighted-unifrac-distance-matrix-[column of data that you want to evaluate].qzv

:::
```
qiime diversity beta-group-significance --i-distance-matrix BETA-unweighted-unifrac-distance-matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column Sulfide_Treatment_nucleic_acid_time_point --p-method permanova --p-pairwise --o-visualization BETA-unweighted-unifrac-distance-matrix-Sulfide_Treatment_nucleic_acid_time_point.qzv

```
:::
Now, run these statistical analyses using the weighted unifrac metric. 
Why? Because Weighted UniFrac also accounts for the relative abundance of lineages between communities

:::info
Could you describe and explain the differences between the beta diversity results summarized in these two graphs?

**unweigthed**
![](https://i.imgur.com/Q2WNxty.png)




**weighted**
![](https://i.imgur.com/XAhdRk7.png)


:::


## Visualizing Beta Diversity in PCo graphs (Emperor)

Principle Coordinates Analysis is a useful tool to represent multidimensional data such as distance matrices in two or three dimensions.  PCOA generates a two (or three) dimensional graph that represents the multidimensional distance between samples.  

To generate a PCOA matrix in qiime:
:::success
qiime diversity pcoa \
&nbsp;  --i-distance-matrix BETA-jaccard_distance_matrix.qza\
&nbsp;  --o-pcoa BETA-jaccard_pcoa_matrix.qza
:::
```
qiime diversity pcoa --i-distance-matrix BETA-jaccard-distance-matrix.qza --o-pcoa BETA-jaccard-pcoa-matrix.qza
```
Now we visualize the pcoa matrix as a plot:
:::success
qiime emperor plot \
&nbsp;--i-pcoa BETA-jaccard_pcoa_matrix.qza \
&nbsp;--m-metadata-file sample-metadata.tsv \
&nbsp;--o-visualization BETA-jaccard-pcoa-plot.qzv
:::
```
qiime emperor plot --i-pcoa BETA-frequency-jaccard-pcoa-matrix.qza --m-metadata-file sample-metadata.tsv --o-visualization BETA-jaccard-pcoa-plot.qzv
```


![](https://i.imgur.com/Ik2eua9.png)

1. Each dot = a 2D representation of one sample and the entire microbial community!
2. Use the drop-down menu to analyze different metadata variables 

Change the column to select different metadata variables and look at the results… what trends do you see in the data?

**Now do the same for the unifrac distances**

```
qiime diversity pcoa --i-distance-matrix BETA-unweighted-unifrac-distance-matrix.qza --o-pcoa BETA-unweighted-unifrac-pcoa-matrix.qza

qiime emperor plot --i-pcoa BETA-unweighted-unifrac-pcoa-matrix.qza --m-metadata-file sample-metadata.tsv --o-visualization BETA-unweighted-unifrac-pcoa-plot.qzv
```

:::info
Now, run the weighted-unifrac metric. 
Can you explain these graphs?
(unweighted left, weighted right)
![](https://i.imgur.com/2Pl60w3.png)
:::

## Using core metrics diversity statistics in QIIME2 (rarefied data)
There is a build option to generate multiple visualizations at once. It will use the normalization by rarefied we learned before. Now that we have run some stats, we can take advantage of this option to run multiple ones in a single command. 

:::success
qiime diversity core-metrics-phylogenetic \
&nbsp;  --i-phylogeny rooted-tree.qza \
&nbsp;  --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza \
&nbsp;  --p-sampling-depth 20000 \
&nbsp;  --m-metadata-file sample-metadata.tsv \
&nbsp;  --output-dir core-metrics-results
:::
Note: use the same --p sampling depth as you did before in the rarify command!!
```
qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza   --i-table table-filtered-no-mitochondria-no-chloroplast-rarefied.qza --p-sampling-depth 20000   --m-metadata-file sample-metadata.tsv --output-dir core-metrics-results

```
Now run the alpha-group-significance for each of the alpha diversity measures. 

```
qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/observed_features_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/observed-features-group-significance.qzv

qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/evenness_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/evenness-group-significance.qzv

qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/faith_pd_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/faith_pd-group-significance.qzv

qiime diversity alpha-group-significance  --i-alpha-diversity core-metrics-results/shannon_vector.qza   --m-metadata-file sample-metadata.tsv   --o-visualization core-metrics-results/shannon-group-significance.qzv

```
And now, for the beta diversity. Remember to select the right  --m-metadata-column to run statistical tests with different variables, these variables should reflect your hypothesis: Does microbial community structure differ significantly among different samples according to X (site, vegetation…)? 

REMEMBER CHANGING [column] to the variable you want to test!

```
qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/unweighted-unifrac-[column]-significance.qzv  --p-pairwise

qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/weighted_unifrac_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/weighted-unifrac-[column]-significance.qzv  --p-pairwise

qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/bray_curtis_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/bray_curtis-[column]-significance.qzv  --p-pairwise

qiime diversity beta-group-significance  --i-distance-matrix core-metrics-results/jaccard_distance_matrix.qza  --m-metadata-file sample-metadata.tsv  --m-metadata-column [column]  --o-visualization core-metrics-results/jaccard-[column]-significance.qzv  --p-pairwise

```


![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 

