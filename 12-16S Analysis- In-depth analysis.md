[TOC]
# Part II. In-depth analysis

![](https://i.imgur.com/sxiUUl8.png)


## Differential abundance ANCOM for identification of taxa of interest
[tutorial](
https://docs.qiime2.org/2021.8/tutorials/pd-mice/?highlight=gneiss#differential-abundance-with-ancom)


Many microbiome investigators are interested in testing whether individual ASVs or taxa are more or less abundant in different sample groups. This is known as differential abundance. Microbiome data present several challenges for performing differential abundance using conventional methods. Microbiome abundance data are inherently sparse (have a lot of zeros) and compositional (everything adds up to 1). Because of this, traditional statistical methods that you may be familiar with, such as ANOVA or t-tests, are not appropriate for performing differential abundance tests of microbiome data and lead to a high false-positive rate. ANCOM is a compositionally aware alternative that allows testing for differentially abundant features.

Before we begin, we will filter out low abundance/low prevalence ASVs. Filtering can provide better resolution and limit false discovery rate (FDR) penalty on features that are too far below the noise threshold to apply to a statistical test. A feature that shows up with 10 counts could be a real feature that is present only in that sample; a feature that’s present in several samples but only got amplified and sequenced in one sample because PCR is a somewhat stochastic process; or it may be noise. It’s not possible to tell, so feature-based analysis may be better after filtering low-abundance features. However, filtering also shifts the composition of a sample, further disrupting the relationship. Here, the filtering is performed as a trade-off between the model, computational efficiency, and statistical practicality.


###ASV analysis
```
qiime feature-table filter-features --i-table table-filtered-no-mitochondria-no-chloroplast.qza  --p-min-frequency 50  --p-min-samples 2 --o-filtered-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund.qza
  ```

ANCOM fundamentally operates on a FeatureTable[Frequency], which contains the frequencies of features in each sample. However, ANCOM cannot tolerate zeros because compositional methods typically use a log transform or a ratio and you can’t take the log or divide by zeros. To remove the zeros from our table, we can add a pseudo count to the FeatureTable[Frequency] Artifact, creating a FeatureTable[Composition] in its place.

```
qiime composition add-pseudocount --i-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund.qza  --o-composition-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund-comp.qza
```

Let’s use ANCOM to check whether there is a difference in the mice based on their donor and then by their genetic background. The test will calculate the number of ratios between pairs of ASVs that are significantly different with FDR-corrected p < 0.05. *Remember to choose the column of interest*.

```
qiime composition ancom --i-table ANCOM-table-filtered-no-mitochondria-no-chloroplast-abund-comp.qza --m-metadata-file sample-metadata.tsv --m-metadata-column nucleic_acid  --o-visualization ANCOM-table-filtered-no-mitochondria-no-chloroplast-nucleic_acid.qzv
```
![](https://i.imgur.com/V63pMxx.png)

:::info

Now, run the ANCOM with the collapsed taxonomy.
Can you explain the differences between the graphs?
What are the advantages of the ASV analysis?
and of the taxonomy-based grouping?

![](https://i.imgur.com/eEhQxzn.png)


:::

## Longitudinal analysis

These analyses require at least two-time points for comparison. Full tutorial [here](https://docs.qiime2.org/2019.10/tutorials/longitudinal/).

![](https://i.imgur.com/Ih3c5Px.png)



### Pairwise difference comparisons
Pairwise difference tests determine whether the value of a specific metric changed significantly between pairs of paired samples (e.g., pre- and post-treatment).

This visualizer currently supports the comparison of feature abundance (e.g., microbial sequence variants or taxa) in a feature table, or of metadata values in a sample metadata file. Alpha diversity values (e.g., observed sequence variants) and beta diversity values (e.g., principal coordinates) are useful metrics for comparison with these tests and should be contained in one of the metadata files given as input. In the example below, we will test whether alpha diversity (Shannon diversity index) changed significantly between two different time points in the ECAM data according to delivery mode


`--p-state-column` needs to be numerical. (usually time)
`--p-metric` needs to be numerical.
`--p-individual-id-column` needs to be categorial (this is what you are testing, the treatment). This column identifies the samples that belong to the same individual through time
`--p-baseline` first time point. (lowest value in `--p-state-column` )

```
qiime longitudinal pairwise-differences   --m-metadata-file sample-metadata.tsv   --m-metadata-file ALPHA-shannon-vector.qza   --p-metric shannon_entropy   --p-group-column Sulfide_Treatment   --p-state-column time_point   --p-state-1 1   --p-state-2 18   --p-individual-id-column nucleic_acid   --p-replicate-handling random   --o-visualization pairwise-differences-nucleic_acid.qzv
```

![](https://i.imgur.com/YxmbiCO.png)

### Pairwise distances comparisons
The pairwise-distances visualizer also assesses changes between paired samples from two different “states”, but instead of taking a metadata column or artifact as input, it operates on a distance matrix. You can find distance matrices in the core-metric-results folder. 



```
qiime longitudinal pairwise-distances  --m-metadata-file sample-metadata.tsv   --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza      --p-group-column Sulfide_Treatment_nucleic_acid --p-state-column time_point   --p-state-1 1   --p-state-2 18   --p-individual-id-column nucleic_acid   --p-replicate-handling random   --o-visualization pairwise-distances-unweighted_unifrac_distance_matrix.qzv
```

### Volatility analysis
The volatility visualizer generates interactive line plots that allow us to assess how volatile a dependent variable is over a continuous, independent variable (e.g., time) in one or more groups. Multiple metadata files (including alpha and beta diversity artifacts) and FeatureTable[RelativeFrequency] tables can be used as input, and in the interactive visualization we can select different dependent variables to plot on the y-axis.

Here we examine how variance in Shannon diversity and other metadata changes across time (set with the state-column parameter) in the ECAM cohort, both in groups of samples (interactively selected as described below) and in individual subjects (set with the individual-id-column parameter).
`--p-individual-id-column column identifying the samples that belong to the same individual through time`

We can look at changes in the beta diversity indexes
```
qiime longitudinal volatility --m-metadata-file sample-metadata.tsv --m-metadata-file BETA-frequency-unweighted-unifrac-pcoa-matrix.qza --p-state-column time_point  --p-individual-id-column Sulfide_Treatment --p-default-group-column Sulfide_Treatment_nucleic_acid_time_point --p-default-metric 'Axis 2'  --o-visualization VOL_Sulfide_Treatment_nucleic_acid_time_point_unweight_frequency_pc_vol.qzv

```

![](https://i.imgur.com/lg2xiwc.png)

or to changes, for example in the number of taxa observed. 

```
 qiime longitudinal volatility --m-metadata-file sample-metadata.tsv --m-metadata-file ./core-metrics-results/observed_features_vector.qza --p-state-column time_point  --p-individual-id-column Sulfide_Treatment --p-default-group-column Sulfide_Treatment_nucleic_acid_time_point   --o-visualization VOL_Sulfide_Treatment_nucleic_acid_time_point_observed_feature_pc_vol.qzv
```

:::success

*How to run weighted-unifrac* 

```

qiime diversity beta-phylogenetic --i-table table-filtered-no-mitochondria-no-chloroplast-frequency.qza --i-phylogeny rooted-tree.qza --p-metric weighted_unifrac --o-distance-matrix BETA-weighted-unifrac-distance-matrix.qza


qiime diversity beta-group-significance --i-distance-matrix BETA-weighted-unifrac-distance-matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column Sulfide_Treatment_nucleic_acid_time_point --p-method permanova --p-pairwise --o-visualization BETA-weighted-unifrac-distance-matrix-Sulfide_Treatment_nucleic_acid_time_point.qzv

qiime diversity pcoa --i-distance-matrix BETA-weighted-unifrac-distance-matrix.qza --o-pcoa BETA-fweighted-unifrac-pcoa-matrix.qza

qiime emperor plot --i-pcoa BETA-weighted-unifrac-pcoa-matrix.qza --m-metadata-file sample-metadata.tsv --o-visualization BETA-weighted-unifrac-pcoa-plot.qzv

qiime longitudinal volatility --m-metadata-file sample-metadata.tsv --m-metadata-file BETA-weighted-unifrac-pcoa-matrix.qza --p-state-column time_point  --p-individual-id-column Sulfide_Treatment --p-default-group-column Sulfide_Treatment_nucleic_acid_time_point --p-default-metric 'Axis 2'  --o-visualization VOL_Sulfide_Treatment_nucleic_acid_time_point_unweight_frequency_pc_vol.qzv

```



:::


![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 


