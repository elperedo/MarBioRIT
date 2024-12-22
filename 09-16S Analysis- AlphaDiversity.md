[TOC]

## Part I. Statistical Analyses

In microbiome experiments, investigators frequently wonder about things like:

* How many different species/OTUs/ASVs are present in my samples?
* How much phylogenetic diversity is present in each sample?
* How similar/different are individual samples and groups of samples?
* What factors (e.g., pH, elevation, blood pressure, body size, or host species just to name a few examples) are associated with differences in microbial composition and biodiversity?

And more. These questions can be answered by alpha- and beta-diversity analyses. Alpha diversity measures the level of diversity within individual samples. Beta diversity measures the level of diversity or dissimilarity between samples. We can then use this information to statistical test whether alpha diversity is different between groups of samples (indicating, e.g., that those groups have more/less species richness) and whether beta diversity is greater between groups (indicating, e.g., that samples within a group are more similar to each other than those in another group, suggesting that membership within these groups is shaping the microbial composition of those samples).
![](https://i.imgur.com/0glFIdz.png)
## Alpha Diversity

There are many ways to express the observed alpha diversity of a sample and the estimated richness of the population based on the abundance of taxa (OTUs, ASVs) in the sample.  The [Shannon](https://en.wikipedia.org/wiki/Diversity_index#Shannon_index) and Simpson Diversity Indeces are two common ways of expressing the richness and relative abundance of taxa observed in the sample; Chao1 is a common nonparametric estimator total richness.  
To see a complete list of available metrics:
```
qiime diversity alpha --help
```
:::info
Additional information on 
[alpha and beta diversity metrics](https://forum.qiime2.org/t/alpha-and-beta-diversity-explanations-and-commands/2282)

:::

And each is defined [here](http://scikit-bio.org/docs/latest/generated/skbio.diversity.alpha).
e.g. 
**Shannon’s index**: Calculates richness and diversity using a natural logarithm. Accounts for both abundance and evenness of the taxa present

First, we generate the diversity metric; for the Shannon index the syntax would be:



:::success
qiime diversity alpha \
&nbsp;   --i-table  table-filtered-no-mitochondria-no-chloroplast.qza \
&nbsp;   --p-metric shannon \
&nbsp;   --o-alpha-diversity ALPHA-shannon-vector.qza 
:::
```
qiime diversity alpha --i-table  table-filtered-no-mitochondria-no-chloroplast.qza --p-metric shannon --o-alpha-diversity ALPHA-shannon-vector.qza
```

Next, we will generate the visual and statistical output for each result:
:::success
qiime diversity alpha-group-significance \
&nbsp;   --i-alpha-diversity ALPHA-shannon-vector.qza \
&nbsp;   --m-metadata-file sample-metadata.tsv \
&nbsp;   --o-visualization ALPHA-shannon-group-significance.qzv
:::
```
qiime diversity alpha-group-significance --i-alpha-diversity ALPHA-shannon-vector.qza --m-metadata-file sample-metadata.tsv --o-visualization ALPHA-shannon-group-significance.qzv 
```
:::info

Do the active (RNA) and present (DNA) microbial communities differ in these samples?

![](https://i.imgur.com/Tej3LTA.png)

:::


Now. Rerun these commands with at least one different metric. e.g. `simpson` or `chao1` in place of `shannon.`  Note that `chao1` returns the 95% confidence interval around the Chao1 estimate.  


![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 


