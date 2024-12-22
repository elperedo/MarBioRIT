# Taxonomic assignment
For many experiments, investigators aim to identify the organisms that are present in a sample. We can do this by comparing our query sequences (i.e., our features, be they ASVs or OTUs) to a reference database of sequences with known taxonomic composition. Simply finding the closest alignment is not really good enough — because other sequences that are equally close matches or nearly as close may have different taxonomic annotations. So we use taxonomy classifiers to determine the closest taxonomic affiliation with some degree of confidence or consensus (which may not be a species name if one cannot be predicted with certainty!), based on alignment, k-mer frequencies, etc.

![](https://i.imgur.com/4GxuhNm.png)


## Acquiring databases for Taxonomical Assigment

We need to download the reference taxonomy databases that we will be using to identify the taxonomy of our ASV
silva, https://www.arb-silva.de/
green genes, https://greengenes.secondgenome.com/

We are going to use SILVA. We are going to download a file that contains the info for the region of the 16S we are using. 


Now we will download the file from the command line. 

**very important** your database has to match the version of your qiime2 installation (in this case 2024.2)


There is a variety of databases we can use for taxonomical classification. See [here](https://docs.qiime2.org/2024.2/data-resources/). Today, we will be using Silva. 
```
wget  -O "silva-138-99-nb-classifier.qza" "https://data.qiime2.org/classifiers/sklearn-1.4.2/silva/silva-138-99-nb-classifier.qza"
```
## Classifying the sequences

Now, we are ready to classify the sequences 


q2-feature-classifier contains three different classification methods. classify-consensus-blast and classify-consensus-vsearch are both alignment-based methods that find a consensus assignment across N top hits. These methods take reference database FeatureData[Taxonomy] and FeatureData[Sequence] files directly, and do not need to be pre-trained.

Machine-learning-based classification methods are available through classify-sklearn, and theoretically can apply any of the classification methods available in scikit-learn. These classifiers must be trained, e.g., to learn which features best distinguish each taxonomic group, adding an additional step to the classification process. Classifier training is reference database- and marker-gene-specific and only needs to happen once per marker-gene/reference database combination; that classifier may then be re-used as many times as you like without needing to re-train!

We will be using the classify-sklearn method, with the pre-trained database we downloaded before. 


:::success
qiime feature-classifier classify-sklearn  \
&nbsp;    --i-classifier silva-138-99-nb-classifier.qza   \
&nbsp;    --i-reads rep-seqs.qza   \
&nbsp;    --o-classification taxonomy.qza 
&nbsp;    --p-n-jobs 16
:::

In the terminal type:
```
qiime feature-classifier classify-sklearn  --i-classifier silva-138-99-nb-classifier.qza  --i-reads rep-seqs.qza  --o-classification taxonomy.qza --p-n-jobs 16
```
In the terminal
```
qiime metadata tabulate  --m-input-file taxonomy.qza --o-visualization taxonomy.qzv
```

Scan the taxonomy classifications. Which bacteria do you see? Are any surprising or expected taxa? You can also use the search bar to find specific taxa.

## Visualizing Taxonomy using bar plots.

One of the most popular ways of visualizing the taxonomical composition of a sample is using a bar plot. This has the advantage of allowing us to compare many samples at once. This type of representation is useful to visualize the most abundant groups.  

:::success
qiime taxa barplot \
&nbsp;    --i-table table.qza \
&nbsp;    --i-taxonomy taxonomy.qza \
&nbsp;    --m-metadata-file sample-metadata.tsv \
&nbsp;    --o-visualization taxa-bar-plots.qzv

:::



```
qiime taxa barplot  --i-table table.qza --i-taxonomy taxonomy.qza --m-metadata-file sample-metadata.tsv  --o-visualization taxa-bar-plots.qzv
```

 Open the data in https://view.qiime2.org/ and change the taxonomical level (kingdom, phylum, class, order, family, genus, species) and order the samples (sort in base to different variables included in the metadata file) … what trends do you see in the data?

You should see something like this:
![](https://i.imgur.com/EfcmAi1.png)

**Export ASV table**

You might want to have your data in an annotated excel table. In this tutorial, we are going to create a folder and an excel compatible file for each table we generate. You can export the data as follows. 
:::success
qiime tools export   \
&nbsp;    --input-path table.qza  \
&nbsp;    --output-path ASV-table
:::

```
qiime tools export  --input-path table.qza   --output-path ASV-table

```

Convert the exported table from biom format to tsv

:::success
biom convert \
&nbsp;    -i ASV-table/feature-table.biom \
&nbsp;    -o ASV-table/feature-table.tsv \
&nbsp;    --to-tsv
:::

```
biom convert -i ASV-table/feature-table.biom -o ASV-table/feature-table.tsv --to-tsv

```

## Using taxonomical information to remove eukaryotic sequences


Have you seen any eukaryotic sequences in the data? 
Why are they there? 

(hint. What is the origin of organelles?)

Using the taxonomical information of each of our ASV, let's filter and remove the mitochondrial and chloroplast ASVs and a new feature table that includes only prokaryotic data. 

:::success
qiime taxa filter-table   \
&nbsp;    --i-table table.qza   \
&nbsp;    --i-taxonomy taxonomy.qza   \
&nbsp;    --p-exclude mitochondria,chloroplast   \
&nbsp;    --o-filtered-table table-no-mitochondria-no-chloroplast.qza
:::
```
qiime taxa filter-table --i-table table.qza  --i-taxonomy taxonomy.qza --p-exclude mitochondria,chloroplast  --o-filtered-table table-no-mitochondria-no-chloroplast.qza
```


:::success
qiime feature-table summarize   \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --o-visualization table-no-mitochondria-no-chloroplast.qzv   \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::
```
qiime feature-table summarize  --i-table table-no-mitochondria-no-chloroplast.qza  --o-visualization table-no-mitochondria-no-chloroplast.qzv  --m-sample-metadata-file sample-metadata.tsv
  
```

Now let's visualize the new table without chloroplast and mitochondria. 
We will use the same scripts we used after dada2, just modifying the input table, and creating summary tables of clustered sequences.

Type:


:::success
qiime feature-table summarize   \
&nbsp;    --i-table table-no-mitochondria-no-chloroplast.qza   \
&nbsp;    --o-visualization table-no-mitochondria-no-chloroplast.qzv   \
&nbsp;    --m-sample-metadata-file sample-metadata.tsv
:::

```
qiime feature-table summarize  --i-table table-no-mitochondria-no-chloroplast.qza  --o-visualization table-no-mitochondria-no-chloroplast.qzv   --m-sample-metadata-file sample-metadata.tsv
```

## Generate new bar plots after the removal of eukaryotic sequences
 
 :::success
 qiime taxa barplot   \
 &nbsp; --i-table table-no-mitochondria-no-chloroplast.qza  \
 &nbsp; --i-taxonomy taxonomy.qza   \
 &nbsp; --m-metadata-file sample-metadata.tsv  \
 &nbsp; --o-visualization taxa-bar-plots-no-mitochondria-no-chloroplast.qzv
 :::
 
 ``` 
 qiime taxa barplot   --i-table table-no-mitochondria-no-chloroplast.qza   --i-taxonomy taxonomy.qza   --m-metadata-file sample-metadata.tsv   --o-visualization taxa-bar-plots-no-mitochondria-no-chloroplast.qzv
 
 ```
 
 Open the data in https://view.qiime2.org/ and change the taxonomical level (kingdom, phylum, class, order, family, genus, species) and order the samples (sort in base to different variables included in the metadata file) … do you see any differences when comparing with the unfiltered file?

Remember that from the visualization file, you will be able to export an excel file of the data at the taxonomical level of your choice. 
You should see something like this:
![](https://i.imgur.com/EfcmAi1.png)


**Export ASV table**

Now, we have a filtered ASV table that can be used in other analyses, e.g. differential abundance tests or for doing Venn diagrams. 
```
qiime tools export   --input-path table-no-mitochondria-no-chloroplast.qza  --output-path ASV-table-no-mitochondria-no-chloroplast
```

Convert the exported table from biom format to tsv
```
biom convert -i ASV-table-no-mitochondria-no-chloroplast/feature-table.biom -o ASV-table-no-mitochondria-no-chloroplast/feature-table.tsv --to-tsv
```

![](https://i.imgur.com/Z6Be0ec.png)

See: https://docs.qiime2.org/



https://forum.qiime2.org/t/tutorial-integrating-qiime2-and-r-for-data-visualization-and-analysis-using-qiime2r/4121 

